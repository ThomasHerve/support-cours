# Linux - Surveillance en Temps Réel avec Zabbix et Nagios

![Tutorial Cover](assets/zabbix-nagios.jpg)

## Introduction

Dans un environnement informatique moderne, la surveillance des systèmes est essentielle pour garantir leur bon fonctionnement. Ce guide vous présentera deux outils puissants : **Zabbix** et **Nagios**. Vous apprendrez à les installer, à les configurer et à les utiliser pour surveiller vos infrastructures en temps réel.

---

## Pourquoi utiliser Zabbix et Nagios ?

### Zabbix
- **Solution complète** : Surveille les serveurs, applications, bases de données et équipements réseau.
- **Alertes avancées** : Configurez des seuils pour recevoir des notifications.
- **Interface graphique intuitive** : Tableaux de bord personnalisables.

### Nagios
- **Fiabilité éprouvée** : Utilisé dans des milliers d'organisations.
- **Flexibilité** : Surveillance des services locaux et distants grâce à des plugins.
- **Modulaire** : Extensible avec des scripts personnalisés.

---

## Partie 1 : Pré-requis avec Docker Compose

Pour faciliter l'installation et la configuration, vous pouvez utiliser **Docker Compose** pour déployer rapidement Zabbix et Nagios.

### Étape 1.1 : Installer Docker et Docker Compose

1. **Installez Docker** :
   ```bash
   sudo apt update
   sudo apt install -y docker.io
   ```

2. **Installez Docker Compose** :
   ```bash
   sudo apt install -y docker-compose
   ```

---

### Étape 1.2 : Fichier Docker Compose

Créez un fichier `docker-compose.yml` contenant les services pour Zabbix et Nagios :

```yaml
version: '3.8'

services:
  zabbix-server:
    image: zabbix/zabbix-server-mysql:latest
    container_name: zabbix-server
    environment:
      DB_SERVER_HOST: zabbix-mysql
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_password
      MYSQL_DATABASE: zabbix
    depends_on:
      - zabbix-mysql
    ports:
      - "10051:10051"
    networks:
      - zabbix-net

  zabbix-web:
    image: zabbix/zabbix-web-apache-mysql:latest
    container_name: zabbix-web
    environment:
      DB_SERVER_HOST: zabbix-mysql
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_password
      MYSQL_DATABASE: zabbix
    ports:
      - "8080:8080"
    depends_on:
      - zabbix-server
    networks:
      - zabbix-net

  zabbix-mysql:
    image: mysql:5.7
    container_name: zabbix-mysql
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: zabbix
      MYSQL_USER: zabbix
      MYSQL_PASSWORD: zabbix_password
    networks:
      - zabbix-net
    volumes:
      - zabbix-mysql-data:/var/lib/mysql

  nagios:
    image: jasonrivers/nagios:latest
    container_name: nagios
    ports:
      - "8081:80"
    environment:
      NAGIOSADMIN_USER: nagiosadmin
      NAGIOSADMIN_PASS: nagios_password
    volumes:
      - nagios-config:/opt/nagios/etc
      - nagios-plugins:/opt/Custom-Nagios-Plugins

networks:
  zabbix-net:

volumes:
  zabbix-mysql-data:
  nagios-config:
  nagios-plugins:
```

---

### Étape 1.3 : Démarrer les Services

1. Lancez Docker Compose dans le répertoire contenant le fichier `docker-compose.yml` :
   ```bash
   docker-compose up -d
   ```

2. Vérifiez que les services sont en cours d'exécution :
   ```bash
   docker ps
   ```

3. Accédez aux interfaces web :
   - **Zabbix** : `http://<votre_IP>:8080`
   - **Nagios** : `http://<votre_IP>:8081`

!!! info 
    Les identifiants par défaut pour **Nagios** sont définis dans le fichier Docker Compose (ex. `nagiosadmin` / `nagios_password`).

---

## Partie 2 : Utilisation des Outils

### Étape 2.1 : Ajouter un Hôte dans Zabbix

1. Accédez à l'interface web de Zabbix.
2. Naviguez vers **Configuration > Hosts** et cliquez sur **Create Host**.
3. Configurez :
   - **Host name** : Nom de l'hôte.
   - **Interfaces** : Adresse IP ou hostname.
   - **Templates** : Associez un modèle de surveillance.
4. Cliquez sur **Add** pour valider.

---

### Étape 2.2 : Surveiller un Service dans Nagios

1. Modifiez le fichier de configuration des services :
   ```bash
   sudo docker exec -it nagios bash
   nano /opt/nagios/etc/objects/localhost.cfg
   ```

2. Ajoutez un service pour surveiller un site web via le protocole HTTP :
   ```text
   define service {
       use                     generic-service
       host_name               localhost
       service_description     HTTP
       check_command           check_http
   }
   ```

3. **Ajout d'un Plugin Personnalisé : Exemple avec le Ping**  
   Les plugins de Nagios permettent de surveiller une grande variété de paramètres et services. Si le plugin souhaité n'existe pas par défaut, vous pouvez en installer un ou le créer vous-même. Prenons l'exemple de la surveillance de la connectivité réseau avec `check_ping`.

   - **Télécharger les Plugins Nagios Officiels** :  
     ```bash
     wget https://nagios-plugins.org/download/nagios-plugins-2.3.3.tar.gz
     tar -xvzf nagios-plugins-2.3.3.tar.gz
     cd nagios-plugins-2.3.3
     ./configure
     make
     sudo make install
     ```
     Ces plugins incluent des scripts pour surveiller des services tels que le ping, le CPU, la mémoire, etc.

   - **Configurer un Nouveau Service pour le Ping** :  
     Ajoutez dans le fichier `localhost.cfg` :
     ```text
     define service {
         use                     generic-service
         host_name               localhost
         service_description     Ping
         check_command           check_ping!100.0,20%!500.0,60%
     }
     ```
     - `100.0,20%` : Tolère un délai moyen inférieur à 100 ms ou une perte de paquets de 20 %.
     - `500.0,60%` : Génère une alerte critique pour des délais moyens supérieurs à 500 ms ou une perte de paquets au-delà de 60 %.

4. Redémarrez Nagios pour appliquer les modifications :
   ```bash
   docker restart nagios
   ```

---

## Partie 3 : Exercices Pratiques

### Exercice 1 : Créer une Alerte dans Zabbix

1. Créez une alerte personnalisée pour surveiller **l'utilisation de la mémoire vive** :
   - Naviguez vers **Configuration > Templates**.
   - Modifiez un modèle et ajoutez un **Item** pour surveiller l'utilisation mémoire en pourcentage :
     - **Name** : Memory Usage.
     - **Key** : `vm.memory.size[available]`.
     - **Type** : Zabbix agent.
     - **Units** : %.
   - Ajoutez un **Trigger** pour générer une alerte si l'utilisation dépasse 80 % :
     - **Name** : High Memory Usage.
     - **Expression** : `{Template OS Linux:vm.memory.size[available].last()}<20`.
2. Simulez une surcharge mémoire :
   ```bash
   stress --vm 2 --vm-bytes 512M --timeout 10s
   ```
3. A vous de jouer ! Choisissez vous meme un parametre à surveiller et implémentez une autre alerte. 

!!! info
    ça peut etre n'importe quoi, google est votre ami


---

### Exercice 2 : Ajouter un Plugin Personnalisé dans Nagios

1. Téléchargez et installez les plugins Nagios comme expliqué ci-dessus.
2. Créez un script personnalisé pour surveiller un service spécifique (par exemple, vérifier qu’un fichier existe) :
   ```bash
   echo '#!/bin/bash
   if [ -f "/tmp/testfile" ]; then
       echo "File exists - OK"
       exit 0
   else
       echo "File does not exist - CRITICAL"
       exit 2
   fi' > /opt/Custom-Nagios-Plugins/check_file.sh
   chmod +x /opt/Custom-Nagios-Plugins/check_file.sh
   ```
3. Configurez le service dans Nagios :
   ```text
   define command {
       command_name    check_file
       command_line    /opt/Custom-Nagios-Plugins/check_file.sh
   }
   ```
4. Ajoutez le service au fichier `localhost.cfg` :
   ```text
   define service {
       use                     generic-service
       host_name               localhost
       service_description     File Check
       check_command           check_file
   }
   ```
5. Redémarrez Nagios :
   ```bash
   docker restart nagios
   ```
!!! info 
    Créez ou supprimez le fichier `/tmp/testfile` pour tester différents états du service.

---

## Conclusion

Zabbix et Nagios sont des outils indispensables pour assurer la surveillance et la disponibilité des infrastructures informatiques. En combinant leurs capacités, vous pouvez surveiller efficacement vos systèmes et réagir rapidement en cas de problème. 🎉
