---
title: "Les bases de Dockerfile"
date: 2023-08-19T12:25:00+02:00
draft: false
---

Pour rappel, une image Docker est un _template_ contenant des instructions pour créer un container Docker.
Un Dockerfile permet de définir ces instructions afin de créer une image Docker.

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
