# Linus - Dockerisation d'un Projet Angular

![Tutorial Cover](assets/docker-angular.jpg)

## Introduction

Dans ce tutoriel, nous allons apprendre à **contenir un projet Angular avec Docker**.  
L'objectif est de partir d'un projet Angular existant et de créer une **image Docker** permettant de l'exécuter facilement.

### Ce que nous allons faire :
1. **Créer un projet Angular (si besoin)**.
2. **Écrire un Dockerfile pour builder et servir l'application**.
3. **Construire l'image Docker et exécuter le conteneur**.
4. **Tester l'application Angular dans un navigateur**.

---

## Partie 1 : Prérequis

Avant de commencer, assurez-vous d’avoir **Node.js** installé sur votre machine :

### Vérification de Node.js :
```bash
node -v
```
Sortie attendue :
```bash
vXX.XX.XX
```

Si Node.js n'est pas installé, consultez [Installation de Node.js](https://nodejs.org/)

---

## Partie 2 : Création d'un Projet Angular

Si vous avez déjà un projet Angular, passez à la **Partie 3**.

### Étape 2.1 : Installer Angular CLI
```bash
npm install -g @angular/cli
```

### Étape 2.2 : Créer un Projet Angular
```bash
ng new mon-projet-angular
cd mon-projet-angular
```
Sélectionnez :
- **CSS** pour les styles.
- **Pas de routing** (ou **oui** selon votre besoin).

### Étape 2.3 : Lancer le Serveur Angular en Local
```bash
ng serve --host 0.0.0.0
```
L'application est accessible sur :
```
http://localhost:4200
```

!!! info
    À ce stade, l'application fonctionne, mais elle dépend de **Node.js**. Nous allons maintenant la dockeriser.

---

## Partie 3 : Dockerisation d'Angular

### Étape 3.1 : Générer les fichiers de build Angular

---

### Étape 3.1 : Écrire le Dockerfile

Dans le dossier du projet, créez un fichier **Dockerfile** :
```bash
nano Dockerfile
```

Ajoutez ce contenu :
```dockerfile
FROM node:alpine

WORKDIR /usr/src/app

COPY . /usr/src/app

RUN npm install -g @angular/cli

RUN npm install

CMD ["ng", "serve", "--host", "0.0.0.0"]
```
---

## Partie 4 : Construction et Exécution du Conteneur

### Étape 4.1 : Construire l’Image Docker
```bash
docker build -t mon-angular .
```

!!! info
    Docker va télécharger les dépendances et compiler Angular. Cela peut prendre du temps la première fois.

### Étape 4.2 : Lancer le Conteneur
```bash
docker run -d -p 8080:4200 --name angular_app mon-angular
```
---

## Partie 5 : Tester l'Application Dockerisée

### Étape 5.1 : Vérifier si le Conteneur Tourne
```bash
docker ps
```
Sortie attendue :
```bash
CONTAINER ID   IMAGE        PORTS                  NAMES
xxxxxxxxxxxx   mon-angular  0.0.0.0:8080->80/tcp   angular_app
```

### Étape 5.2 : Ouvrir l'Application
Allez sur :
```
http://localhost:8080
```
Vous devriez voir l'application Angular fonctionner !

!!! info
    Si l’application ne s’affiche pas, vérifiez que le conteneur est bien lancé avec `docker ps`.

---

## Exercice Pratique

### Objectif
1. **Créer une variante du Dockerfile** qui utilise `http-server` (au lieu de Nginx).
2. **Construire et exécuter l’image**.
3. **Tester si l’application s’affiche toujours sur `http://localhost:8081`**.

---

