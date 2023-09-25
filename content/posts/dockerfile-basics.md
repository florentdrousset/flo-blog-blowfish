---
title: "Les bases de Dockerfile - Partie I"
date: 2023-08-19T12:25:00+02:00
draft: false
---

## Préambule

Pour rappel :
* une image Docker est un _template_ contenant des instructions pour créer un container Docker
* un fichier Dockerfile permet de définir ces instructions afin de créer une image Docker
* une image Docker peut comporter plusieurs layers et commence toujours avec un layer de base

Pour notre premier test, nous allons utiliser une image de la distro Linux [Alpine](https://www.alpinelinux.org/) comme layer de base et y installer vim.
La première instruction de notre Dockerfile (nommé `Dockerfile`) sera donc :

```dockerfile
FROM alpine:latest
```

En effet, une image peut être (et sera la plupart du temps) basée sur une autre image.
Pour ce premier exemple, nous nous basons sur la dernière version de Alpine. Si nous souhaitions utiliser une version spécifique, on pourrait alors écrire `alpine:3.15.10` (par exemple).

La prochaine instruction sera

```dockerfile
RUN apk update && \
apk add vim
```

L'instruction RUN va créer un nouveau _layer_ Docker et exécuter l'instruction passée en paramètre. En l'occurence, les instructions executées seront `apk update`
et `apk add vim`. 

## Construction de l'image

Pour constuire une image à partir de notre Dockerfile, on utilisera `docker build`.
`docker build` requiert à minima un contexte : en l'occurence le dossier dans lequel se trouve le Dockerfile.
On utilisera également le flag `-t` afin de tagger notre image avec un nom pour des raisons de praticité.

```dockerfile
docker build -t dockerfile-tutorial .
```

Docker va donc build notre image. Pour vérifier si l'image a bien été créée :

```dockerfile
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

`-ti`: `-t` alloue un pseudo-TTY : le container sera run avec un terminal interactif. `-i` : Permet de garder _STDIN_ ouvert, afin de pouvoir interagir avec le container en direct.

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

`docker ps` permet de lister tous les containers actifs. Comme nous venons de quitter notre container, celui-ci ne devrait pas apparaître dans cette liste.

## Un mot sur les layers

Chaque instruction de notre Dockerfile construit un nouveau layer au-dessus du précédent.
Ceux-ci sont automatiquement mis en cache par Docker lors du build : Docker essayera toujours de récupérer un layer en cache après avoir vérifié si celui-ci a été modifié.

Si un layer a changé depuis le dernier build, __tous les layers suivants seront rebuild__. L'optimisation du cache dans Docker fera probablement l'objet d'un article à part entière,
mais vous pouvez vous référer à [l'article dédié](https://docs.docker.com/build/cache/) dans la documentation officielle de Docker.

Reprenons le retour console lorsque que nous exécutons la commande `docker build -t dockerfile-tutorial .` :

```
[+] Building 0.4s (7/7) FINISHED
=> [internal] load build definition from Dockerfile                                                                                                                                                    0.0s
=> => transferring dockerfile: 88B                                                                                                                                                                     0.0s
=> [internal] load .dockerignore                                                                                                                                                                       0.0s
=> => transferring context: 2B                                                                                                                                                                         0.0s
=> [internal] load metadata for docker.io/library/alpine:latest                                                                                                                                        0.4s
=> [1/3] FROM docker.io/library/alpine:latest@sha256:7144f7bab3d4c2648d7e59409f15ec52a18006a128c733fcff20d3a4a54ba44a                                                                                  0.0s
=> CACHED [2/3] RUN apk update                                                                                                                                                                         0.0s
=> CACHED [3/3] RUN apk add vim                                                                                                                                                                        0.0s
=> exporting to image                                                                                                                                                                                  0.0s
=> => exporting layers                                                                                                                                                                                 0.0s
=> => writing image sha256:011e7455dac2771e18a365a9e9fae2951eb199c7f0b901b7b2cbb60ef5cdaa68                                                                                                            0.0s
=> => naming to docker.io/library/dockerfile-tutorial
```

On peut constater que 3 layers ont été lancés :
1 - le layer de base : l'image `alpine:latest`
2 - l'instruction `RUN apk update` (récupérée du cache)
3 - l'instruction `RUN apk add vim` (également récupérée du cache)

Avoir une bonne compréhension des layers est important pour comprendre la construction des images Docker.

## Vers des images plus complexes

Bien, nous savons désormais nous baser sur une image pré-existante avec l'instruction `FROM` et exécuter des commandes avec l'instruction `RUN`.
Prenons un nouvel exemple et essayons de nous lancer dans une création d'image plus complexe : l'installation du framework PHP [Symfony](https://symfony.com/).

Voici le fichier Dockerfile complet :

```dockerfile
FROM php:8.0-apache

RUN apt-get update && apt-get install -y \
libpng-dev \
libjpeg62-turbo-dev \
libxml2-dev \
libicu-dev \
git \
unzip

RUN docker-php-ext-configure gd --with-jpeg=/usr/include/ \
&& docker-php-ext-install gd pdo pdo_mysql opcache intl

RUN a2enmod rewrite

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

WORKDIR /var/www/html

RUN composer create-project symfony/skeleton my_project_name

ENV APACHE_DOCUMENT_ROOT /var/www/html/my_project_name/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf

EXPOSE 80

CMD ["apache2-foreground"]
```

Sont ici présentes 4 instructions Dockerfile que nous n'avons pas encore vues : `WORKDIR`, `ENV`, `EXPOSE` et `CMD`.

Reprenons les différentes instructions une par une.

Nous commencons par nous baser sur l'image officielle de php-apache.
Les deux `RUN` suivants installent toutes les librairies requises pour l'installation de Symfony, suivi d'une modification de config du serveur Apache (RUN a2enmod rewrite).

`WORKDIR` définit simplement le dossier dans lequel les commandes suivantes seront exécutées. Il est l'équivalent du `cd`, à ceci près qu'il créera le dossier (donc également équivalent du `mkdir`) si celui-ci n'existe pas déjà !

`ENV` permet de définir une variable d'environnement qui pourra être réutilisée par la majeure partie des instructions Dockerfile.

```dockerfile
ENV APACHE_DOCUMENT_ROOT /var/www/html/my_project_name/public
```

Une autre syntaxe existe : `ENV nom_de_variable="valeur"`

Ici, la variable d'environnement `APACHE_DOCUMENT_ROOT` définie après `ENV` est réutilisée comme ceci : `${APACHE_DOCUMENT_ROOT}`.
_Note_ : dans certains cas, on préfèrera utiliser l'instruction `ARG` qui permet de définir des variables utilisées uniquement au moment du build et qui ne seront pas persistées dans le container.

`EXPOSE` informe Docker que le container devra écouter sur le port spécifié. `EXPOSE` est configuré en TCP par défaut, mais on peut lui spécifier d'utiliser le protocole UDP si nécessaire :

```dockerfile
EXPOSE 80/udp
```

`CMD` définit la commande executée par défaut lorsque le container est lancé.
Il ne peut y avoir qu'une seule instruction `CMD` par Dockerfile ! (Notez que si plusieurs sont présentes, seule la dernière sera executée.)
Il est possible de surcharger `CMD` au moment du run avec la syntaxe suivante : `docker run $image $autre_commande`

Et voilà ! On peut donc build notre image, puis lancer le container :

```
docker build -t test-symfony .
docker run -p 8080:80 test-symfony
```

`-p` nous permet de spéficier les ports utilisés par notre container. Le premier nombre définit le port utilisé physiquement sur votre machine, et le lie au deuxième nombre qui définit le port utilisé dans le container.
Rendez-vous sur localhost:8080 pour voir la page d'accueil de Symfony.

Cet article n'était qu'une introduction aux fichiers Dockerfile qui ne fait qu'effleurer les possibilités de la construction d'image Docker. La deuxième partie suivra bientôt pour aborder d'autres notions importantes.