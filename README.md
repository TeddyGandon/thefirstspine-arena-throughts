# Réflexions techniques et architecturelles autour du jeu de société en ligne

## Introduction

Ce document de travail a été rédigé avant et pendant le développement de The First Spine Arena, déclinaison digitale du jeu The First Spine. Il présente les aspects techniques fondamentaux du jeu sous forme de réflexions techniques.

Les rélfexions présentées ici découlent de nombreuses expérience dans la conception de jeux sur de nombreuses plateformes, avec de nombreuses technologies.

## Sandbox et boucles de gameplay

Il est aisé de définir un jeu sur ses boucles de gameplay. Ainsi, dans TFS, il existe une boucle générale par joueur, et d'autres plus petites sur ses actions possibles. Pourtant, lors du développement d'un jeu en version numérique, il faut voir le jeu comme un ensemble de règles, un "bac à sable" où ses actions sont définies par des restrictions et non plus des possibilités.

Cela évite l'écueil classique de :

```
Si le joueur fait ça, alors
Mais si le joueur fait ça, alors
Mais si le joueur fait ça, alors
...
```

ou de :

```
Si le joueur en est à cette phase de jeu :
    - Si le joueur fait ça, alors
    - Mais si le joueur fait ça, alors
    - Mais si le joueur fait ça, alors
    ...
Mais si le joueur en est à cette phase de jeu :
    - Si le joueur fait ça, alors
    - Mais si le joueur fait ça, alors
    - Mais si le joueur fait ça, alors
    ...
...
```

Il s'agit du problème majeur, selon moi, des solutions pré-existantes qui existent sur Internet. Au lieu de voir les jeux sur leur plateforme comme un ensemble de règles indépendantes, ces offres proposent un modèle où le jeu est enfermé dans sa boucle de gameplay sans jamais pouvoir en sortir. Après tout, nombre de jeux cassent leurs propres boucles afin de créer la surprise.



## Actions, priorisation

TODO

## Situations changeantes

TODO
