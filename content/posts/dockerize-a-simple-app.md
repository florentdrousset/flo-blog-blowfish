---
title: "Dockerizer une app web simple"
date: 2023-09-24T18:54:50+02:00
draft: false
---

## Introduction

Dans ce petit tutoriel, nous allons explorer comment Docker peut simplifier le déploiement d'une application web. Plus précisément, nous allons dockeriser une application web existante, `httpbin`, et y ajouter une couche de mise en cache avec Redis, le tout orchestré par docker compose.

## Prérequis

- Docker installé sur votre machine
- Connaissances de base en ligne de commande

## Étapes du Tutoriel

### 1. Dockerisation de httpbin

Tout d'abord, créons un fichier `Dockerfile` (voir mon [intro à Dockerfile](../dockerfile-basics) pour vous rafraîchir la mémoire !) avec le contenu suivant :

```Dockerfile
# Nous utiliserons une image python officielle comme image parent
FROM python:3.9-slim

# On définit le répertoire de travail dans le container
WORKDIR /usr/src/app

# On installe httpbin et gunicorn
RUN pip install httpbin gunicorn

# On définit ici la commande à exécuter lors du démarrage du conteneur
CMD ["gunicorn", "-b", "0.0.0.0:80", "httpbin:app"]
```

## 2. Utilisation de Docker Compose
Créez un fichier docker-compose.yml :


```Dockerfile
version: '3'

services:
  httpbin:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:80"
  redis:
    image: "redis:latest"
    ports:
      - "6379:6379"
```

Lancez les services avec Docker Compose :

```
docker-compose up
```

## 3.  Implémenter la Persistance des Données
Ajoutez un volume à la définition de service Redis dans docker-compose.yml :

```Dockerfile
redis:
image: "redis:latest"
ports:
- "6379:6379"
volumes:
- redis-data:/data
```

Créez un volume Docker à coup de `docker volume create redis-data`.
Un volume Docker est une forme de stockage persistant que vous pouvez utiliser avec vos conteneurs Docker.
Contrairement à un dossier mappé sur l'hôte, un volume est complètement géré par Docker et permet de sauvegarder des données indépendamment du cycle de vie des conteneurs. Dans notre cas, nous avons utilisé un volume pour stocker les données de notre base de données Redis, afin de ne pas les perdre lorsque le conteneur est arrêté ou supprimé.


#### Tester httpbin

Vous pouvez tester le service `httpbin` de plusieurs manières :

1. **Navigateur Web** : Ouvrez `http://localhost:8080` dans votre navigateur.

2. **Curl** : Vous pouvez utiliser `curl` pour envoyer des requêtes HTTP :

```bash
curl http://localhost:8080/get
```

#### Tester Redis

1. **Shell Redis** :

```bash
docker exec -it [CONTAINER_ID] redis-cli
```

Puis, une fois dans le shell :

```bash
set user:1 "John Doe"
get user:1
```

Cela va créer une clé `user:1` avec la valeur `"John Doe"` et ensuite récupérer cette valeur.

2. **Script ou application** : Vous pouvez également utiliser un script Python ou une application pour interagir avec Redis.

```python
import redis

r = redis.Redis(host='localhost', port=6379, db=0)
r.set('user:2', 'Jane Doe')
print(r.get('user:2').decode("utf-8"))
```


