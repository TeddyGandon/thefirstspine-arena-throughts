# Réflexions techniques et architecturelles autour du jeu de société en ligne

## Introduction

Ce document de travail a été rédigé avant et pendant le développement de The First Spine Arena, déclinaison digitale du jeu The First Spine. Il présente les aspects techniques fondamentaux du jeu sous forme de réflexions techniques.

Les rélfexions présentées ici découlent de nombreuses expérience dans la conception de jeux sur de nombreuses plateformes, avec de nombreuses technologies.

## Sandbox et boucles de gameplay

Cette réflexion est plus scémantique qu'autre chose, et présente une base pour toutes les autres réflexions présentent dans ce document.

### Le problème

Il est aisé de définir un jeu sur ses boucles de gameplay. Ainsi, dans TFS, il existe une boucle générale par joueur, et d'autres plus petites sur ses actions possibles. Pourtant, lors du développement d'un jeu en version numérique, il faut voir le jeu comme un ensemble de règles, un "bac à sable" où ses actions sont définies uniquement par possibilités.

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

### Voir le jeu comme un bac à sable

En prenant du recul sur un jeu et en le voyant comme un ensemble de possibilités, il est plus aisé de sortir de conditions infernales. TFS a été développé avec un seul mot d'ordre :

```
Le joueur à fait ça, il peut désormais faire ça.
```

Une différence qui n'est somme toute pas flagrante scémentiquement, mais qui en développement change tout :

```
if ($this->state === GameStates::PHASE_1)
{
   if ($this->player->did(Actions::ACTION_A)
   {
      // Le serveur fait quelque chose
   }
   else if ($this->player->did(Actions::ACTION_B)
   {
      // Le serveur fait quelque chose
   }
   else
   {
       // Action non pris en charge
   }
}
else if ($this->state === GameStates::PHASE_2)
{
   if ($this->player->did(Actions::ACTION_C)
   {
      // Le serveur fait quelque chose
   }
   else if ($this->player->did(Actions::ACTION_D)
   {
      // Le serveur fait quelque chose
   }
   else
   {
       // Action non pris en charge
   }
}
else
{
    // Phase non pris en charge
}
```

devient alors :

```
$action; // l'action qu'à effectué le joueur
$actionWorker = Worker::factory($action);
return $actionWorker->possibleActions();
```

## Actions, priorisation, conflits

TODO

## Hookes et exceptions

## Situations changeantes

TODO

### La loi de la coquille vide

TODO

### Le serveur a toujours raison

TODO
