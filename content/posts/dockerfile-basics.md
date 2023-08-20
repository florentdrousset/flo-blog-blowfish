---
title: "Les bases de Dockerfile"
date: 2023-08-19T12:25:00+02:00
draft: false
---

Pour rappel :
* une image Docker est un _template_ contenant des instructions pour créer un container Docker
* un fichier Dockerfile permet de définir ces instructions afin de créer une image Docker
* unne image Docker peut comporter plusieurs layers et commence toujours avec un layer de base

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

