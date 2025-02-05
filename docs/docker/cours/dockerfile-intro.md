# Introduction à l'Écriture d'un Dockerfile

![Tutorial Cover](assets/dockerfile.jpg)

## Introduction

Un **Dockerfile** est un fichier texte contenant une série d’instructions permettant de **créer une image Docker personnalisée**. Il définit tout ce dont un conteneur a besoin pour fonctionner, comme :
- Le **système de base** (ex. Debian, Ubuntu, Alpine...).
- Les **dépendances** et les **logiciels** nécessaires.
- Les **commandes exécutées au démarrage**.

Dans ce tutoriel, nous allons :  
1. **Créer un Dockerfile** simple.  
2. **Construire une image Docker** avec ce Dockerfile.  
3. **Lancer un conteneur basé sur cette image**.  
4. **Tester son bon fonctionnement**.  

---

## Partie 1 : Structure de Base d'un Dockerfile

Un Dockerfile est composé d’instructions clés. Voici un exemple basique :

```dockerfile
# 1. Définir l’image de base
FROM ubuntu:latest

# 2. Auteur de l’image
LABEL maintainer="Votre Nom <email@example.com>"

# 3. Mettre à jour le système et installer des paquets
RUN apt update && apt install -y curl

# 4. Définir la commande par défaut à exécuter
CMD ["echo", "Hello depuis mon conteneur personnalisé !"]
```

Explication des instructions :
- `FROM ubuntu:latest` → Utilise **Ubuntu** comme image de base.
- `LABEL maintainer="Votre Nom"` → Ajoute une **étiquette** avec les informations de l’auteur.
- `RUN apt update && apt install -y curl` → Met à jour le système et installe **curl**.
- `CMD ["echo", "Hello depuis mon conteneur personnalisé !"]` → Affiche un message par défaut.

---

## Partie 2 : Création d'un Dockerfile et Construction d'une Image

Nous allons maintenant **créer et tester** notre Dockerfile.

### Étape 2.1 : Créer un répertoire de travail
```bash
mkdir mon_dockerfile && cd mon_dockerfile
```

### Étape 2.2 : Créer le Dockerfile
```bash
nano Dockerfile
```
Collez le contenu suivant :
```dockerfile
FROM ubuntu:latest
LABEL maintainer="Votre Nom <email@example.com>"
RUN apt update && apt install -y curl
CMD ["echo", "Hello depuis mon conteneur personnalisé !"]
```

### Étape 2.3 : Construire l’image Docker
```bash
docker build -t mon_image_personnalisee .
```

Explication :
- `-t mon_image_personnalisee` → Donne un nom à l’image (`mon_image_personnalisee`).
- `.` → Utilise le **Dockerfile** dans le dossier actuel.

!!! info
    Vous verrez plusieurs étapes s’exécuter lors de la construction de l’image.

---

## Partie 3 : Lancer et Tester le Conteneur

Une fois l’image créée, lançons un conteneur basé sur celle-ci.

### Étape 3.1 : Démarrer un conteneur
```bash
docker run --name mon_conteneur mon_image_personnalisee
```

Sortie attendue :
```bash
Hello depuis mon conteneur personnalisé !
```

!!! info
    Le conteneur exécute la commande `echo` définie dans le Dockerfile, puis s’arrête.

### Étape 3.2 : Vérifier la Liste des Conteneurs
```bash
docker ps -a
```

Sortie attendue :
```bash
CONTAINER ID   IMAGE                   COMMAND   STATUS                     NAMES
xxxxxxx        mon_image_personnalisee "echo ..." Exited (0) xx seconds ago mon_conteneur
```

!!! info
    Comme notre conteneur exécute uniquement `echo`, il **s’arrête immédiatement** après avoir affiché le message.

---

## Partie 4 : Cas Pratique - Un Serveur Web avec un Fichier HTML

Nous allons maintenant créer une **image avec Nginx** qui servira une **page HTML personnalisée**.

### Étape 4.1 : Créer un nouveau répertoire et fichier HTML
```bash
mkdir mon_serveur_web
cd mon_serveur_web
echo "<h1>Bienvenue sur mon serveur web Docker !</h1>" > index.html
```

### Étape 4.2 : Créer un Dockerfile
```bash
nano Dockerfile
```
Ajoutez ce contenu :
```dockerfile
# Utiliser l’image Nginx officielle
FROM nginx:latest

# Copier le fichier HTML personnalisé dans le conteneur
COPY index.html /usr/share/nginx/html/index.html

# Exposer le port 80
EXPOSE 80

# Lancer Nginx
CMD ["nginx", "-g", "daemon off;"]
```

!!! info
    Nous pouvons constater deux nouveaux mots clé:
    COPY: Permet de copier un fichier ou un dossier local **dans l'image**  
    EXPOSE: Indique que le port doit être ouvert

### Étape 4.3 : Construire et Lancer l’Image
```bash
docker build -t mon_nginx .
docker run -d -p 8080:80 --name serveur_web mon_nginx
```

### Étape 4.4 : Tester le Serveur Web
Ouvrez votre navigateur et allez à l’URL :
```
http://localhost:8080
```

!!! info
    Vous devriez voir le message `Bienvenue sur mon serveur web Docker !`.

---

## Exercice Pratique

### Objectif :
1. **Créer un Dockerfile** qui génère une image basée sur Debian.
2. **Installer Apache (`apache2`)** et copier un fichier HTML personnalisé dans ce même dockerfile.
3. **Lancer le conteneur et tester l’accès depuis le navigateur**.

---
