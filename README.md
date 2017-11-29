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

Voir le jeu comme un bac à sable offrant uniquement des possibilités plutôt que comme un ensemble de restrictions permet de déléguer les effets des actions des joueurs à des entités indépendantes et non restrictives. Notons aussi que cette vision oublie complètement les phases de jeux et ses boucles de gameplay, autorisant ainsi une possibilité d'action qui n'a pas été spécifiquement prévue par les règles.

## Actions, priorisation, conflits

Comment traiter une action ? Comment délivrer les possibilités aux joueurs ? Comment traiter les actions prioritaires ?

### Actions, réponses et possibilités

Nous avons vu comment était traité notre jeu, c'est à dire en un ensemble de possibilités. Ces possibilités sont des actions dont le jeu attend une réponse.

```
Réponse à une possibilité -> JEU -> Livrer les possibilités suivantes
```

Dans ce schéma, le joueur répond à une possibilité offerte par le jeu, et le jeu délivre d'autres possibilités. Ces dernières sont appelées "actions".

Nombre de jeux permettent plusieurs actions à faire à la suite ou non suivant son type et sa complexité. Un jeu complexe va offrir un nombre important d'actions (Scythes, pour ne pas le citer) tandis qu'un jeu simple va donner moins d'actions au joueur (Mysterium par exemple). Attention toutefois : un jeu simple ne veut pas forcément dire qu'il n'a aucune profondeur, et l'inverse est aussi vrai.

Par exemple, dans TFS les possibilités offertes sont simples et peu variées :
- Défausser et/ou piocher
- Jouer un sort et/ou déplacer une créature et/ou placer une carte
- Résoudre les confrontations

Mais il y a un problème et pas des moindres : certaines actions necessitent plusieurs réponses de la part de l'utilisateur. Déplacer une créature requiert du joueur qu'il choisisse une créature, et qu'il choisisse une case à côté de cette même créature. Solution : découper les actions possibles en un ensemble d'instructions simples.

### Les instructions

Les instructions d'une action doivent rester simples et suivre le même schéma de possibilités qu'une action. Par exemple, une instruction simple pourrait être :

```
Choisir une carte sur le plateau de jeu de type "créature"
```

L'ensemble des instructions pour déplacer une créature sur le plateau de jeu pourraient donc être :

```
Choisir une carte sur le plateau de jeu de type "créature"
Choisir une case vide à côté de cette carte
```

Envoyer l'action au jeu au lieu de ces instructions simples présentent un avantage non négligeable : pouvoir annuler l'action en cours et en choisir une autre. Une fois que les instructions sont remplies, le joueur envoie la réponse à l'action avec le résulat des instructions de cette action.

On a donc :

![alt](/actions.svg)

### Le nerf de la guerre

Parfois, le jeu peut demander une action supplémentaire selon le contexte. Et parfois, le contexte impose au joueur de choisir parmi plusieurs actions.

Le jeu doit donc classer les actions par priorité, et envoyer au joueur uniquement les actions les plus prioritaires et qui ont la même priorité.

Par exemple, sur ces actions

```
Action à priorité de 1    =>    à garder et ne pas envoyer
Action à priorité de 1    =>    à garder et ne pas envoyer
Action à priorité de 1    =>    à garder et ne pas envoyer
Action à priorité de 2    =>    à garder et ne pas envoyer
Action à priorité de 3    =>    à envoyer
Action à priorité de 1    =>    à garder et ne pas envoyer
Action à priorité de 3    =>    à envoyer
```

le jeu va demander au joueur de répondre à l'une des actions de priorité **3**. Le jeu va tout de même garder les actions de priorité **1** et **2** pour les proposer plus tard à l'utilisateur. Ce système de "priorisation par paquets" résoud la plupart des conflits que le jeu pourrait rencontrer lorsque, par exemple, plusieurs effets d'une action précédente entrent en contradiction avec l'autre, une action pouvant supprimer une autre action ou pouvant aussi en créer avec une priorité différente.

Autre avantage non négligeable : ces actions ne respectent par à la lettre les usages de "tours" et de "phase de jeu". Ainsi, une action avec une priorité élevée pourra être demandée à un utilisateur pendant le tour du joueur actuel.

### Hooks et exceptions

Le problème des actions contradictoires étant régler, certains effets peuvent générer des actions sans aucune action de l'utilisateur. C'est le cas par exemple lorsque le texte d'une carte commence par

>Au début de votre tour, ...

Certains effets peuvent se résoudre également lors d'un certain contexte, attendant que l'utilisateur effectue une action sans être pour autant explicite. Comme exemple, nous pouvons donner

>Lorsque vous jouez un sortilège, ...

Il serait là aussi aisé de créer des exceptions dans le jeu au moment où le joueur lance un sort ou lorsqu'il commence son tour. Mais n'oublions pas la sandbox : nous énumérons les possibilités et non pas les restrictions. Et il existe en programmation un concept qui permet d'effectuer des executions sans pour autant modifier chaque traitement de réponse : les hooks.

>Un hook (littéralement « crochet » ou « hameçon ») permet à l'utilisateur d'un logiciel de personnaliser le fonctionnement de ce dernier, en lui faisant réaliser des actions supplémentaires à des moments déterminés. Le concepteur du logiciel prévoit des hooks au long du fonctionnement de son programme, qui sont des points d'entrée vers des listes d'actions. Par défaut, le hook est généralement vide et seules les fonctionnalités de base de l'application sont exécutées. Cependant, l'utilisateur peut « accrocher » des morceaux de programme à ces hooks pour personnaliser le logiciel.
— Wikipedia (ze référence) : https://fr.wikipedia.org/wiki/Hook_(informatique)

Il est donc plus facile de traiter ces exceptions comme étant des "anomalies" selon le contexte.

![alt](/hooks.svg)

## Le problème du client quantique

TODO

### La loi de la coquille vide

TODO

### Le serveur a toujours raison

TODO
