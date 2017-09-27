+++
author = "Christian Kakesa"
categories = ["C++", "Programmation"]
date = "2017-10-01"
description = ""
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
title = "C++ std::vector::emplace_back VS std::vector::push_back"
type = "post"
draft = "true"

+++

![Logo ISOCPP](/images/logo_cpp_w260.png#center)

Vous avez sans doute déjà lu un programme **C++** qui utilisait **`std::vector::emplace_back`** et vous vous demandiez quelle est la différence avec `std::vector::push_back` ?

Je me pose la même question. À travers un mini sujet, nous allons voir quelles sont les différences entre **`std::vector::emplace_back`** et `std::vector::push_back`.

Bizarrement, je vais exposer ma conclusion et laisser ceux qui ont le temps de lire le déroulé de mon sujet.

Préférez utiliser **`std::vector::emplace_back`** qui crée les instances à l'endroit où elles doivent être stockées, sans utiliser de déplacement ou des copies intermédiaires d'objets.

# Le petit sujet en C++

Pour tester ces 2 méthodes on va créer une structure **Image** qui contient 2 membres : **name** et **type**.
Pour suivre les copies et déplacements des instances, on va implémenter le construsteur par copie et le constructeur par déplacement.
Vous remarquerez que pour le constructeur par déplacement : `Image(const Image&& image) noexcept`, j'utilise le mot clé **noexcept** qui remplit les conditions nécessaires pour que `std::vector` utilise le constructeur par déplacement plutôt que celui par copie, consommateur en temps et en mémoire.

## Avec std::vector.emplace_back

Avec `std::vector::emplace_back`, pour stocker nos **3 objets** dans notre conteneur, on utilise 3 fois le constructeur par déplacement ; en optimisant le code avec `images.reserve(3)`, **on ne fait pas appel au constructeur par déplacement** et l'instance est créée directement à l'emplacement prévu. Lorsqu'on fournit le nombre d'élément dont on a besoin, le programme n'a pas besoin de réservé des objets vides dans notre conteneur (`std::vector::capacity`).
Ça semble parfait.

```bash
# Résultat avec std::vector::reserve

```

```bash
# Résultat sans std::vector::reserve
Moved!
Moved!
Moved!
```

```cpp
#include <iostream>
#include <string>
#include <vector>

struct Image {
  std::string name;
  std::string type;
  
  Image(std::string name, std::string type = "png"): name(name), type(type) {}
  Image(const Image& image): name(image.name), type(image.type) {
    std::cout << "Copied!" << std::endl;
  }
  Image(const Image&& image) noexcept : name(std::move(image.name)), type(std::move(image.type)) {
    std::cout << "Moved!" << std::endl;
  }
};

int main() {
  std::vector<Image> images;
  images.reserve(3);
  images.emplace_back("destiny_2.png", "png");
  images.emplace_back("transformers_forged-to-fight.png", "png");
  images.emplace_back("cat.gif", "gif");

  return EXIT_SUCCESS;
}

```

## Avec std::vector.push_back

Avec `std::vector::push_bak`, pour stocker nos **3 objets**, on utilise **6 fois le constructeur par déplacement**.
<i>Kezako</i> !
En plus des instances temporaires utilisées dans le contexte de la fonction `int main() {}`, des instances sont préparées pour se loger dans les slots alloués dynamiquement par `std::vector::capacity`.

```bash
# Résultat
Moved!
Moved!
Moved!
Moved!
Moved!
Moved!
```

```cpp
#include <iostream>
#include <string>
#include <vector>

struct Image {
  std::string name;
  std::string type;
  
  Image(std::string name, std::string type = "png"): name(name), type(type) {}
  Image(const Image& image): name(image.name), type(image.type) {
    std::cout << "Copied!" << std::endl;
  }
  Image(const Image&& image) noexcept : name(std::move(image.name)), type(std::move(image.type)) {
    std::cout << "Moved!" << std::endl;
  }
};

int main() {
  std::vector<Image> images;
  images.push_back({"destiny_2.png", "png"});
  images.push_back({"transformers_forged-to-fight.png", "png"});
  images.push_back({"cat.gif", "gif"});
  
  return EXIT_SUCCESS;
}

```