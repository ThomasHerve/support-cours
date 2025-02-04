# Linus - Surveillance des Conteneurs Docker avec Zabbix

![Tutorial Cover](assets/zabbix-docker.jpg)

## Introduction

Zabbix est un puissant système de surveillance open-source permettant de suivre la disponibilité, les performances et l'intégrité de vos systèmes et applications. Dans ce tutoriel, nous allons utiliser **Docker Compose** pour déployer une instance de Zabbix et surveiller un conteneur exécutant un petit service. L'objectif est de comprendre comment configurer Zabbix pour surveiller un conteneur et interpréter les résultats lorsqu'il est **up** (actif) ou **down** (inactif).

---

## Pré-requis

Avant de commencer, vous devez avoir :

- Docker et Docker Compose installés sur votre machine.
- Une connaissance de base du fonctionnement de Docker et Docker Compose.
- Un conteneur Docker en cours d'exécution à surveiller.

---

## Partie 1 : Installation de Zabbix avec Docker Compose

### Étape 1.1 : Créer le fichier `docker-compose.yml`

1. Créez un répertoire pour votre projet Zabbix :
   ```bash
   mkdir zabbix-docker
   cd zabbix-docker
   ```

2. Créez un fichier `docker-compose.yml` dans ce répertoire :
  ```yaml
  services:
    postgresql-server:
      image: postgres:latest
      container_name: postgresql-server
      restart: unless-stopped
      environment:
        POSTGRES_USER: zabbix
        POSTGRES_PASSWORD: password
        POSTGRES_DB: zabbix
      volumes:
        - postgresql-data:/var/lib/postgresql/data

    zabbix-server:
      image: zabbix/zabbix-server-pgsql:latest
      container_name: zabbix-server
      restart: unless-stopped
      depends_on:
        - postgresql-server
      environment:
        DB_SERVER_HOST: postgresql-server
        POSTGRES_USER: zabbix
        POSTGRES_PASSWORD: password
        POSTGRES_DB: zabbix
      ports:
        - "10051:10051"
      volumes:
        - zabbix-server-data:/var/lib/zabbix
        - zabbix-snmptraps-data:/var/lib/zabbix/snmptraps
        - zabbix-export-data:/var/lib/zabbix/export

    zabbix-web-nginx-pgsql:
      image: zabbix/zabbix-web-nginx-pgsql:latest
      container_name: zabbix-web
      restart: unless-stopped
      depends_on:
        - postgresql-server
        - zabbix-server
      environment:
        DB_SERVER_HOST: postgresql-server
        POSTGRES_USER: zabbix
        POSTGRES_PASSWORD: password
        POSTGRES_DB: zabbix
        ZBX_SERVER_HOST: zabbix-server
        PHP_TZ: Europe/London
      ports:
        - "8080:8080"
      volumes:
        - zabbix-web-data:/usr/share/zabbix

    zabbix-agent:
      image: zabbix/zabbix-agent:latest
      container_name: zabbix-agent
      restart: unless-stopped
      depends_on:
        - zabbix-server
      environment:
        ZBX_HOSTNAME: "zabbix-server"
        ZBX_SERVER_HOST: zabbix-server
        ZBX_SERVER_PORT: '10051'
        ZBX_SERVER_ACTIVE: zabbix-server
      expose:
        - 10050

    my-app:
      image: nginx:alpine
      container_name: my-app
      ports:
        - "8081:80"


  volumes:
    postgresql-data:
    zabbix-server-data:
    zabbix-snmptraps-data:
    zabbix-export-data:
    zabbix-web-data:

  ```

### Étape 1.2 : Lancer Zabbix avec Docker Compose

1. Démarrez les conteneurs avec Docker Compose :
   ```bash
   docker-compose up -d
   ```

2. Vérifiez que les conteneurs sont en cours d'exécution :
   ```bash
   docker ps
   ```

   Vous devriez voir les conteneurs suivants : `zabbix-server`, `zabbix-web`, `zabbix-db`, et `my-app`.

---

## Partie 2 : Accéder à l'Interface Web de Zabbix

1. Ouvrez votre navigateur et accédez à l'interface web de Zabbix via l'URL suivante :
   ```text
   http://localhost:8080
   ```

2. Connectez-vous avec les identifiants par défaut :
   - **Utilisateur** : `Admin`
   - **Mot de passe** : `zabbix`

---

## Partie 3 : Ajouter un Hôte (Conteneur) à Zabbix

### Étape 3.1 : Monitorer l'host

Nous allons déjà commencer par nous monitorer nous même. Cela est sensé être natif, mais comme nous passons par docker nous avons une petite modification à faire.
1. Allez dans **Monitoring** -> **Hosts** et cliquez sur **Zabbix-server** -> **Host** (sous **configuration**)
2. Dans **Agent** enlevez l'ip et mettez dans **DNS name** `zabbix-agent` (le dns docker de notre agent).
3. Passez le **Connect to** à `DNS`.
4. Cliquez sur **Update**.
5. Attendez une minute et verifiez dans **Dashboard**

### Étape 3.2 : Créer le template de monitoring

Zabbix fonctionne par template. Par defaut vous en avez une grande quantité disponible, par exemple nous pourrions utiliser celui de nginx et suivre les guideline du template pour que note nginx puisse remonter des metrics à zabbix. Ici nous allons à l'inverse créé un template très simple pour verifier si le conteneur répond en `HTTP`.

1. Allez dans **Data collection** -> **Templates** et cliquez sur **Create template**.
2. Remplissez les champs comme suit :
   - **Template name** : `Template App HTTP`
   - **Template groups** : `Templates` (celui par defaut)

3. Appuyez sur `Add`
4. Cherchez votre template nouvellement créé via le système de filtre et cliquez sur **Items**
5. Appuyez sur **Create Item**
6. Remplissez les champs comme suit :
   - **Name** : `HTTP Check`
   - **Type** : `HTTP agent` (celui par defaut)
   - **Key**: web.page.get
   - **Type of information**: Text
   - **URL**: my-app:80
7. Vous pouvez tester la bonne execution grace au bouton **Test** puis vous pouvez sauvegarder l'item.


### Étape 3.3 : Ajouter le Conteneur à Zabbix

1. Allez dans **Monitoring** -> **Hosts** et cliquez sur **Create host**.
2. Remplissez les champs comme suit :
   - **Host name** : `my-app`
   - **Groups** : Vous pouvez créer un groupe "Docker" ou utiliser "Linux servers".
   - **Template** : `Template App HTTP`

3. Cliquez sur **Add** pour enregistrer l'hôte.

## Étape 3.3 : Ajouter un trigger

1. Allez dans **Monitoring** -> **Hosts** et séléctionnez `my-app`
2. Cliquez sur **Triggers**
3. Cliquez sur **Create trigger**
4. Remplissez les champs comme suit :
   - **Name** : `my-app trigger`
   - **Severity**: `High`
   - **Expression** : `nodata(/my-app/web.page.get,10)=1` (Nous verifions si nous avons de la donnée depuis les 10 dernières secondes)
5. CLiquez sur **Add**
---

## Partie 4 : Vérifier le Statut du Conteneur

### Étape 4.1 : Accéder aux Données de Surveillance

1. Allez dans **Monitoring** -> **Latest data**.
2. Sélectionnez l'hôte `my-app` et vous verrez des métriques telles que l'état du service HTTP et la disponibilité de l'application.

### Étape 4.2 : Résultats Attendus

- **Quand le conteneur est UP (actif)** :
  - De la donnée coté latest data et aucun problèmes

- **Quand le conteneur est DOWN (inactif)** :
  - Un problème doit etre visible dans le dashboard principal
---

## Partie 5 : les Notifications Zabbix

Nous n'allons pas couvrir cette partie dans le cours car nous ne sommes pas équiper pour, mais sachez qu'en entreprise vous pourriez envoyer des mails via un serveur smtp par exemple.

---

## Exercice Pratique

### Objectif : Configurer un nouveau service à surveiller dans un autre conteneur Docker

1. Créez un nouveau service Docker à surveiller (par exemple, un serveur Apache ou Redis).
2. Ajoutez ce nouveau conteneur à Zabbix comme un hôte.
3. Configurez un template pertinent pour ce service (par exemple, **Template App Redis** pour Redis ou **Template App HTTP** pour Apache).
4. Testez la surveillance de ce service en démarrant et en arrêtant le conteneur.

---

## Conclusion

Dans ce tutoriel, nous avons configuré Zabbix pour surveiller un conteneur Docker exécutant un service, utilisé Docker Compose pour gérer les services et testé la surveillance de l'état de l'hôte via Zabbix. Vous avez appris à configurer des hôtes, ajouter des templates et tester la surveillance en simulant des pannes. Zabbix est un outil puissant pour centraliser la surveillance de vos conteneurs et services.
