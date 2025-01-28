# Linus - Configuration et Utilisation de Rsyslog

![Tutorial Cover](assets/rsyslog.jpg)

## Introduction

Rsyslog est un démon puissant et flexible pour la gestion des journaux système sur les serveurs Linux. Il permet de centraliser, d’analyser et de transférer les journaux provenant de diverses applications et systèmes. Ce guide explique comment installer, configurer et utiliser Rsyslog efficacement, avec des exemples pratiques.

---

## Pourquoi utiliser Rsyslog ?

- **Centralisation des logs** : Rsyslog peut regrouper les journaux de plusieurs machines sur un serveur central.
- **Flexibilité** : Les journaux peuvent être filtrés, triés et formatés.
- **Compatibilité** : Supporte divers protocoles (TCP, UDP, RELP) et formats de logs.
- **Évolutivité** : Gère un grand volume de journaux dans des environnements complexes.

---

## Partie 1 : Installation de Rsyslog

### Étape 1.1 : Vérification de l'installation

Rsyslog est souvent installé par défaut sur les systèmes modernes. Vérifiez sa présence avec :
```bash
rsyslogd -v
```
Si Rsyslog est installé, la version s'affiche.

---

### Étape 1.2 : Installer Rsyslog (si nécessaire)

1. Installez Rsyslog via votre gestionnaire de paquets :
   ```bash
   # Ubuntu/Debian
   sudo apt update && sudo apt install rsyslog

   # CentOS/RedHat
   sudo yum install rsyslog

   # SUSE
   sudo zypper install rsyslog
   ```

2. Démarrez et activez le service :
   ```bash
   sudo systemctl start rsyslog
   sudo systemctl enable rsyslog
   ```

3. Vérifiez l’état du service :
   ```bash
   sudo systemctl status rsyslog
   ```

---

## Partie 2 : Configuration de Base de Rsyslog

### Étape 2.1 : Localisation des fichiers de configuration

Le fichier principal de configuration de Rsyslog se trouve ici :
```bash
/etc/rsyslog.conf
```
Les configurations spécifiques peuvent être ajoutées dans :
```bash
/etc/rsyslog.d/*.conf
```

---

### Étape 2.2 : Exemple de Configuration Locale

1. Ouvrez le fichier de configuration principal :
   ```bash
   sudo nano /etc/rsyslog.conf
   ```

2. Activez les modules nécessaires :
   - Assurez-vous que les modules d'entrée sont décommentés :
     ```text
     module(load="imuxsock")  # Logs générés localement
     module(load="imklog")   # Logs du kernel
     ```

3. Ajoutez une règle pour séparer les logs SSH dans un fichier dédié :
   ```text
   if $programname == 'sshd' then /var/log/ssh.log
   & stop
   ```

4. Redémarrez Rsyslog pour appliquer les changements :
   ```bash
   sudo systemctl restart rsyslog
   ```

---

### Étape 2.3 : Tester la Configuration Locale

1. **Générer un log pour SSH** :  
   Essayez de vous connecter en SSH à `localhost` :
   ```bash
   ssh <votre_utilisateur>@localhost
   ```

2. **Vérifiez que le log est enregistré dans `/var/log/ssh.log`** :
   ```bash
   sudo tail -f /var/log/ssh.log
   ```

!!! info 
    Cette méthode permet de confirmer que vos règles Rsyslog fonctionnent correctement.

---

## Partie 3 : Configuration de Rsyslog comme Serveur Central

### Étape 3.1 : Activer la réception des journaux distants

1. Activez le module d’écoute pour les connexions réseau :
   ```bash
   sudo nano /etc/rsyslog.conf
   ```

2. Décommentez les lignes pour activer les protocoles réseau :
   ```text
   module(load="imtcp")  # Activer TCP
   input(type="imtcp" port="514")  # Ecoute sur le port 514
   module(load="imudp")  # Activer UDP
   input(type="imudp" port="514")  # Ecoute sur le port 514
   ```

3. Redémarrez Rsyslog :
   ```bash
   sudo systemctl restart rsyslog
   ```

---

### Étape 3.2 : Configurer un Client pour envoyer des logs

1. Ouvrez la configuration Rsyslog sur le client :
   ```bash
   sudo nano /etc/rsyslog.conf
   ```

2. Ajoutez la ligne suivante pour envoyer les logs au serveur Rsyslog (remplacez `<IP_SERVEUR>` par `127.0.0.1` si vous testez en local) :
   ```text
   *.* @<IP_SERVEUR>:514  # Utilisation d’UDP
   *.* @@<IP_SERVEUR>:514  # Utilisation de TCP
   ```

3. Redémarrez le client Rsyslog :
   ```bash
   sudo systemctl restart rsyslog
   ```

---

### Étape 3.3 : Tester la Configuration Réseau

1. **Générer un log sur le client** :
   ```bash
   logger "Ceci est un test pour Rsyslog"
   ```

2. **Vérifiez sur le serveur que le log a été reçu** :
   ```bash
   sudo tail -f /var/log/syslog
   ```

!!! info 
    Assurez-vous que le port 514 est ouvert sur le serveur pour permettre les connexions.

---

## Partie 4 : Utilisation Avancée de Rsyslog

### Étape 4.1 : Rotation des logs

Pour éviter que les fichiers de logs ne deviennent trop volumineux, configurez la rotation des logs avec `logrotate`.

1. Créez un fichier de configuration pour vos logs personnalisés :
   ```bash
   sudo nano /etc/logrotate.d/rsyslog-custom
   ```

2. Ajoutez les règles de rotation :
   ```text
   /var/log/ssh.log {
       weekly
       rotate 4
       compress
       missingok
       notifempty
       create 640 syslog adm
   }
   ```

3. Testez la configuration :
   ```bash
   sudo logrotate -f /etc/logrotate.d/rsyslog-custom
   ```

---

### Étape 4.2 : Filtres personnalisés

Vous pouvez créer des règles avancées pour filtrer et trier les logs :
- Exemple : Enregistrer les erreurs critiques dans un fichier dédié.
   ```text
   if $syslogseverity <= '3' then /var/log/critical.log
   & stop
   ```

---

## Conclusion

Rsyslog est un outil puissant et polyvalent pour gérer et analyser les journaux système. Il peut être utilisé localement pour organiser les logs ou à grande échelle pour centraliser les journaux dans un environnement distribué. En combinant ses fonctionnalités avec des outils comme ElasticSearch, Rsyslog devient un pilier pour la surveillance et l’analyse des systèmes. 
