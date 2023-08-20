---
title: "Les bases de Dockerfile"
date: 2023-08-19T12:25:00+02:00
draft: false
---

## Préambule

Pour rappel :
* une image Docker est un _template_ contenant des instructions pour créer un container Docker
* un fichier Dockerfile permet de définir ces instructions afin de créer une image Docker
* une image Docker peut comporter plusieurs layers et commence toujours avec un layer de base

Pour notre premier test, nous allons utiliser une image de la distro Linux Alpine et y installer vim.
La première instruction de notre Dockerfile (nommé `Dockerfile`) sera donc :

```
FROM alpine:latest
```

En effet, une image peut être (et sera la plupart du temps) basée sur une autre image.
En l'occurence, nous nous basons sur la dernière version de Alpine. Si nous souhaitions utiliser une version spécifique, on pourrait alors écrire alpine:3.15.10.

La prochaine instruction sera

```
RUN apk update && \
apk add vim
```

L'instruction RUN va créer un nouveau _layer_ Docker et exécuter l'instruction passée en paramètre. En l'occurence, les instructions executées seront `apk update`
et `apk add vim`. 

## Construction de l'image

Pour constuire une image à partir de notre Dockerfile, on utilisera `docker build`.
`docker build` requiert à minima un contexte : en l'occurence le dossier dans lequel se trouve le Dockerfile.
On utilisera également le flag `-t` afin de tagger notre image avec un nom pour des raisons de praticité.

```
docker build -t dockerfile-tutorial .
```

Docker va donc build notre image. Pour vérifier si l'image a bien été créée :

```
docker images
```

Retour console :

```
REPOSITORY                         TAG               IMAGE ID       CREATED         SIZE
dockerfile-tutorial                latest            4f21a4e055d1   40 hours ago    41.4MB
```

## Construction du container

Bien ! Nous avons notre image, nous pouvons à présent créer un container à partir de celle-ci.
Le container sera une instance de notre image qui n'est, encore une fois, qu'un modèle ou __template__ permettant de créer des containers Docker.
Pour run un container à partir de notre image :

```
docker run --rm -ti dockerfile-tutorial /bin/sh
```

Analysons cette commande :

`docker run` : lance un conteneur Docker à partir d'une image spécifiée (en l'occurence, `dockerfile-tutorial`)

`--rm`: indique à Docker de supprimer automatiquement le conteneur une fois qu'il est arrêté. Sans cette option, on devrait manuellement supprimer le conteneur à coup de `docker rm` après l'avoir arrêté.

`-ti`: `-t` Alloue un pseudo-TTY : le container sera run avec un terminal interactif. `-i` : Permet de garder _STDIN_ ouvert, afin de pouvoir interagir avec le container en direct.

Si tout s'est bien passé, vous devriez être gratifié d'un :
```
/ #
```

Vous voilà donc connecté au terminal de votre nouveau container basé sur l'image alpine.
Comme nous avions spécifié l'installation de _vim_ dans le Dockerfile, nous pouvons tester si celui-ci fonctionne bien :
```
vim
```

Nous voilà sur _vim_, lancé depuis notre nouveau container construit à partir du Dockefile que nous avons écrit.
`:q` + entrée pour quitter vim, et taper `exit` pour quitter le container.

`docker ps` permet de lister tous les containers actifs.
Notre container nommé `dockerfile-tutorial` ne devrait pas apparaître dans cette liste, car nous l'avions lancée avec le flag `--rm`.

