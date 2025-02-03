# Linux - Stratégies de Sauvegarde dans Linux avec Docker Compose

![Tutorial Cover](assets/backup-strategies.jpg)

## Introduction

Les **sauvegardes** sont essentielles pour garantir l'intégrité des données et minimiser les pertes en cas de panne, erreur humaine ou cyberattaque. Dans ce cours, nous allons :  

✅ Mettre en place un **environnement Docker** avec une base de données et une application web.  
✅ Configurer un **conteneur de sauvegarde** permettant d’effectuer des exports et des restaurations.  
✅ Expliquer comment déclencher une sauvegarde et une restauration en pratique.  

---

## Partie 1 : Déploiement de l'Environnement

### Étape 1.1 : Création du Fichier `docker-compose.yml`

Créez un dossier de projet et accédez-y :

```bash
mkdir ~/docker-backup && cd ~/docker-backup
```

Créez un fichier `Dockerfile` et ajoutez le contenu:

```yaml
FROM php:apache

# Installer l'extension MySQLi
RUN docker-php-ext-install mysqli

# Activer mod_rewrite (utile pour des frameworks)
RUN a2enmod rewrite
```

Créez un fichier `docker-compose.yml` et ajoutez le contenu suivant :

```yaml
version: '3.8'

services:
  db:
    image: mysql:8
    container_name: db_backup_demo
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: demo_db
      MYSQL_USER: user
      MYSQL_PASSWORD: userpassword
    volumes:
      - db_data:/var/lib/mysql

  app:
    build: .
    container_name: php_app
    restart: always
    volumes:
      - ./app:/var/www/html
    ports:
      - "8080:80"
  
  backup:
    image: mysql:8
    container_name: backup_service
    volumes:
      - ./backup:/backup
    depends_on:
      - db
    entrypoint: tail -f /dev/null  # Permet de garder le conteneur actif pour les backups

volumes:
  db_data:
```

---

### Étape 1.2 : Création de l'Application Web

- Créez un dossier `app` :

   ```bash
   mkdir app
   ```

- Ajoutez un fichier `index.php` dans `app/` :

   ```bash
   nano app/index.php
   ```

3. Ajoutez le code suivant (une interface minimale) :

   ```php
   <?php
   $conn = new mysqli("db", "user", "userpassword", "demo_db");

   if ($conn->connect_error) {
       die("Échec de connexion : " . $conn->connect_error);
   }

   if ($_SERVER["REQUEST_METHOD"] == "POST") {
       $message = $conn->real_escape_string($_POST["message"]);
       $conn->query("INSERT INTO messages (content) VALUES ('$message')");
   }

   $result = $conn->query("SELECT * FROM messages");
   ?>

   <h1>Messages</h1>
   <form method="POST">
       <input type="text" name="message" required>
       <button type="submit">Ajouter</button>
   </form>

   <ul>
       <?php while ($row = $result->fetch_assoc()) { ?>
           <li><?php echo htmlspecialchars($row["content"]); ?></li>
       <?php } ?>
   </ul>
   ```

---

### Étape 1.3 : Lancer l'Environnement

Démarrez les services :

```bash
docker-compose up -d
```

Créez la table MySQL associée :

```bash
docker exec -it db_backup_demo mysql -u root -prootpassword -e "CREATE TABLE demo_db.messages (id INT AUTO_INCREMENT PRIMARY KEY, content TEXT);"
```

Accédez à l’application dans un navigateur :  
👉 **http://localhost:8080**  

Vous pouvez ajouter des messages qui seront stockés dans la base de données.

---

## Partie 2 : Sauvegarde et Restauration de la Base de Données

### Étape 2.1 : Effectuer une Sauvegarde

- Lancez une sauvegarde avec `mysqldump` :

   ```bash
   docker exec backup_service mysqldump -h db -u root -prootpassword --databases demo_db > backup/db_backup.sql
   ```

- Vérifiez le fichier généré :

   ```bash
   ls -lh backup/
   ```

!!! info 
    Le fichier `db_backup.sql` contient toutes les tables et données de `demo_db`.

---

### Étape 2.2 : Simuler une Perte de Données

- Supprimez toutes les données de la table :

   ```bash
   docker exec -it db_backup_demo mysql -h db -u root -prootpassword -e "DELETE FROM demo_db.messages;"
   ```

- Vérifiez que la base est vide :

   ```bash
   docker exec -it db_backup_demo mysql -h db -u root -prootpassword -e "SELECT * FROM demo_db.messages;"
   ```

!!! info 
    À ce stade, la base est vide et les messages ne s'afficheront plus dans l’application.

---

### Étape 2.3 : Restaurer la Base de Données

- Restaurez la base de données avec le fichier de sauvegarde :

   ```bash
   docker exec -i db_backup_demo mysql -h db -u root -prootpassword < backup/db_backup.sql
   ```

- Vérifiez que les données sont restaurées :

   ```bash
   docker exec -it db_backup_demo mysql -h db -u root -prootpassword -e "SELECT * FROM demo_db.messages;"
   ```

!!! info 
    Les messages doivent réapparaître dans la base et sur l’application web.

---

## Partie 3 : Automatiser les Sauvegardes

Vous pouvez planifier une sauvegarde automatique avec **cron**.  

- Ouvrez le fichier crontab :

   ```bash
   crontab -e
   ```

- Ajoutez la ligne suivante pour exécuter une sauvegarde toutes les 12 heures :

   ```text
   0 */12 * * * docker exec backup_service mysqldump -u root -prootpassword --databases demo_db > backup/db_backup.sql
   ```

!!! info 
    Cette tâche enregistrera une copie de la base de données deux fois par jour.

- Exercice: Ajoutez un conteneur qui execute ce cron toute les minutes et constatez sa bonne execution

---

## Conclusion

🎯 **Résumé des étapes** :
1. Déploiement d’un environnement Docker avec une base MySQL et une application web.  
2. Mise en place d’un conteneur de sauvegarde avec `mysqldump`.  
3. Sauvegarde et restauration manuelle des données.  
4. Automatisation des sauvegardes avec cron.  

Cette approche garantit la sécurité des données et permet une restauration rapide en cas d’incident. 🚀
