# Introduction à docker compose

![Tutorial Cover](assets/docker-compose.jpg)

## Introduction

Docker Compose est un outil permettant de **définir et exécuter des applications multi-conteneurs** à l'aide d'un fichier de configuration YAML.  
Au lieu de lancer plusieurs conteneurs manuellement avec `docker run`, Docker Compose **simplifie le processus** en gérant les services, les volumes et les réseaux en une seule commande.

---

## Partie 1 : Installation de Docker Compose

Docker Compose est inclus par défaut avec **Docker Desktop**.  
Sur Linux, il peut être installé séparément.

### Étape 1.1 : Vérifier si Docker Compose est installé
```bash
docker compose version
```
Sortie attendue :
```bash
Docker Compose version vX.X.X
```

### Étape 1.2 : Installer Docker Compose (si absent)
Sur **Ubuntu / Debian** :
```bash
sudo apt update && sudo apt install docker-compose -y
```
Sur **CentOS / RedHat** :
```bash
sudo yum install docker-compose -y
```
Sur **SUSE** :
```bash
sudo zypper install docker-compose
```

---

## Partie 2 : Premier Projet avec Docker Compose

Nous allons créer un fichier `docker-compose.yml` pour exécuter un **serveur web Nginx** avec une page HTML personnalisée.

### Étape 2.1 : Créer l'arborescence du projet
```bash
mkdir mon-projet-compose
cd mon-projet-compose
mkdir html
```

### Étape 2.2 : Ajouter un fichier HTML
```bash
echo "<h1>Bienvenue sur mon serveur Nginx avec Docker Compose !</h1>" > html/index.html
```

### Étape 2.3 : Écrire le fichier `docker-compose.yml`
Créez un fichier `docker-compose.yml` :
```bash
nano docker-compose.yml
```

Ajoutez ce contenu :
```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
```

Explication :
\- **Définit un service `web`** basé sur l’image `nginx:latest`.
\- **Mappe le port `8080`** du PC vers le port `80` du conteneur.
\- **Monte le dossier `html/`** dans `/usr/share/nginx/html` pour servir notre page.

!!! info
    la commande docker equivalente serait `docker run -p 8080:80 -v ./html:/usr/share/nginx/html nginx:latest`

---

## Partie 3 : Lancer et Tester l'Application

### Étape 3.1 : Démarrer les Conteneurs
Dans le dossier contenant `docker-compose.yml`, exécutez :
```bash
docker compose up -d
```
Explication :
- `-d` → Exécute en **mode détaché** (en arrière-plan).

!!! info
    Pour voir les logs des conteneurs en temps réel, utilisez `docker compose logs -f`.

### Étape 3.2 : Vérifier si le Conteneur Tourne
```bash
docker compose ps
```
Sortie attendue :
```bash
NAME             IMAGE       PORTS                  STATUS
mon-projet-web   nginx       0.0.0.0:8080->80/tcp   Up
```

### Étape 3.3 : Tester dans un Navigateur
Allez sur :
```
http://localhost:8080
```
👉 Vous devriez voir : **"Bienvenue sur mon serveur Nginx avec Docker Compose !"**.

---

## Partie 4 : Gestion des Conteneurs avec Docker Compose

### **Arrêter les Conteneurs**
```bash
docker compose down
```
👉 **Arrête et supprime les conteneurs et les réseaux**.

### **Redémarrer les Conteneurs**
```bash
docker compose restart
```

### **Vérifier les Logs**
```bash
docker compose logs
```

---

## Partie 5 : Ajouter une Base de Données avec MySQL

Nous allons maintenant **ajouter une base de données MySQL** à notre `docker-compose.yml`.

### Étape 5.1 : Modifier `docker-compose.yml`
```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html

  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: testdb
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

Explication :
- **Ajout d'un service `db`** basé sur MySQL 5.7.
- **Configuration des variables d'environnement** (mot de passe, utilisateur...).
- **Ajout d'un volume nommé `mysql_data`** pour **stocker les données** de manière persistante.

### Étape 5.2 : Relancer Docker Compose
```bash
docker compose up -d
```

### Étape 5.3 : Vérifier si MySQL fonctionne
```bash
docker compose ps
```
Sortie attendue :
```bash
NAME             IMAGE       PORTS                    STATUS
mon-projet-web   nginx       0.0.0.0:8080->80/tcp     Up
mon-projet-db    mysql:5.7   0.0.0.0:3306->3306/tcp   Up
```

### Étape 5.4 : Tester la Connexion MySQL
```bash
docker exec -it $(docker compose ps -q db) mysql -uuser -ppassword testdb
```
Si tout fonctionne, vous aurez accès au terminal MySQL.

---

## Partie 6: communication inter conteneur

### Étape 6.1 : Modifier `docker-compose.yml`
```yaml
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html

  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: testdb
      MYSQL_USER: user
      MYSQL_PASSWORD: password
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  curl:
    image: alpine/curl:latest
    command: sleep infinity

volumes:
  mysql_data:
```
Explication :
- **Ajout d'un service `curl`**.
- **command: `sleep infinity`** permet de faire en sorte que le conteneur reste en vie.

### Étape 6.2 : interroger le conteneur `web`

Connectez vous au conteneur `curl`
```bash
docker compose exec -it curl sh
```

dans l'invite de commande écrivez:
```bash
curl web
```
Vous devriez voir le contenu de la page html. Nous constatons donc que depuis un conteneur il est possible d'en interroger un autre en utilisant son nom comme url.  

## Exercices Pratique

### Exercice 1
1. **Créez un service du framework backend de votre choix**. 
    Ce dernier doit pouvoir se connecter à la base de donnée et repondre à des requetes REST (exemple: express, fastapi, symfony...)