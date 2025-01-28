# Linux - Analyse des Logs de Sécurité avec Auth.log et Syslog

![Tutorial Cover](assets/authlog-syslog.jpg)

## Introduction

Les journaux de sécurité sont une ressource précieuse pour détecter des connexions suspectes, des tentatives de brute-force ou des activités non autorisées. Ce guide vous explique comment utiliser **auth.log** (le fichier journal dédié aux authentifications) et **Syslog** pour centraliser et analyser ces événements. Vous apprendrez également à configurer, interpréter et exploiter ces logs pour renforcer la sécurité.

---

## Pourquoi analyser auth.log ?

### Auth.log
- **Suivi des authentifications** : Captures des connexions SSH, sudo, échecs de connexion, etc.
- **Détection d'activités suspectes** : Identifier des IP malveillantes ou des tentatives d'accès non autorisées.
- **Dépannage** : Aide à résoudre les problèmes liés aux permissions et aux authentifications.

### Syslog
- **Centralisation des logs** : Permet de regrouper auth.log et d'autres journaux en un seul endroit.
- **Filtrage et analyse** : Facilite l'identification rapide d'incidents critiques.
- **Évolutivité** : Peut être intégré à des outils comme ElasticSearch pour une analyse avancée.

---

## Partie 1 : Configuration et Accès aux Logs

### Étape 1.1 : Vérification des Fichiers Journaux

1. Auth.log est situé dans :
   ```bash
   /var/log/auth.log
   ```

2. Vérifiez son contenu avec :
   ```bash
   sudo tail -f /var/log/auth.log
   ```
!!! info 
    Si vous êtes sur une distribution CentOS ou RedHat, les logs d’authentification sont regroupés dans `/var/log/secure`.

---

### Étape 1.2 : Installer et Configurer Syslog (si nécessaire)

1. Vérifiez si **Rsyslog** est installé :
   ```bash
   rsyslogd -v
   ```

2. Installez-le si nécessaire :
   ```bash
   # Ubuntu/Debian
   sudo apt update && sudo apt install rsyslog

   # CentOS/RedHat
   sudo yum install rsyslog

   # SUSE
   sudo zypper install rsyslog
   ```

3. Activez et démarrez Syslog :
   ```bash
   sudo systemctl start rsyslog
   sudo systemctl enable rsyslog
   ```

---

## Partie 2 : Analyse des Logs avec Auth.log

### Étape 2.1 : Détection de Connexions SSH

1. Ouvrez auth.log pour analyser les connexions SSH :
   ```bash
   sudo grep "sshd" /var/log/auth.log
   ```

2. Points à analyser :
   - **Connexions réussies** :
     ```text
     Accepted password for <user> from <IP> port <port> ssh2
     ```
   - **Tentatives échouées** :
     ```text
     Failed password for invalid user <user> from <IP> port <port> ssh2
     ```

3. Identifier une IP malveillante (ex. brute-force) :
   ```bash
   sudo grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -nr
   ```

!!! info 
    La commande ci-dessus compte et trie les IP ayant effectué des tentatives échouées.

---

### Étape 2.2 : Suivi des Connexions Sudo

1. Recherchez les commandes sudo exécutées :
   ```bash
   sudo grep "sudo:" /var/log/auth.log
   ```

2. Interprétez les logs :
   - **Commande exécutée** :
     ```text
     Jan 28 10:12:00 linus sudo: user : TTY=pts/0 ; PWD=/home/user ; USER=root ; COMMAND=/usr/bin/apt update
     ```
   - **Échecs d'exécution** :
     ```text
     Jan 28 10:13:45 linus sudo: pam_unix(sudo:auth): authentication failure; logname=user
     ```

---

### Étape 2.3 : Détection de Modifications dans Auth.log

1. Surveiller auth.log en temps réel :
   ```bash
   sudo tail -f /var/log/auth.log
   ```

2. Utilisez `watch` pour analyser les changements en continu :
   ```bash
   watch "sudo grep 'Failed password' /var/log/auth.log"
   ```

---

## Partie 3 : Intégration avec Syslog

### Étape 3.1 : Centraliser les Logs Auth avec Syslog

1. Ajoutez une règle pour capturer spécifiquement auth.log :
   ```bash
   sudo nano /etc/rsyslog.d/auth.conf
   ```

2. Insérez les lignes suivantes :
   ```text
   auth,authpriv.*                /var/log/auth_custom.log
   ```

3. Rechargez Syslog pour appliquer la configuration :
   ```bash
   sudo systemctl restart rsyslog
   ```

4. Vérifiez le fichier customisé :
   ```bash
   sudo tail -f /var/log/auth_custom.log
   ```

!!! info 
    Ce fichier dédié simplifie l’analyse des journaux d’authentification.

---

## Partie 4 : Exercices Pratiques

### Exercice 1 : Détection de Tentatives de Brute-Force SSH

1. Lancez une tentative de connexion SSH invalide :
   ```bash
   ssh invalid_user@localhost
   ```

2. Vérifiez l’apparition de l’échec dans auth.log :
   ```bash
   sudo tail -n 10 /var/log/auth.log
   ```

3. Identifiez les IP ayant échoué plusieurs fois :
   ```bash
   sudo grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c
   ```

---

### Exercice 2 : Suivi des Connexions Utilisateur

1. Connectez-vous en SSH :
   ```bash
   ssh your_user@localhost
   ```

2. Surveillez auth.log pour vérifier la connexion réussie :
   ```bash
   sudo tail -f /var/log/auth.log
   ```

3. Filtrez les connexions réussies uniquement :
   ```bash
   sudo grep "Accepted password" /var/log/auth.log
   ```

---

### Exercice 3 : Créez un Journal Personnalisé pour sudo

1. Ajoutez une règle dans Syslog pour les commandes sudo :
   ```bash
   sudo nano /etc/rsyslog.d/sudo.conf
   ```

2. Ajoutez :
   ```text
   :programname, isequal, "sudo" /var/log/sudo.log
   ```

3. Rechargez Syslog :
   ```bash
   sudo systemctl restart rsyslog
   ```

4. Vérifiez les journaux sudo :
   ```bash
   sudo tail -f /var/log/sudo.log
   ```

---

## Conclusion

Les fichiers journaux comme **auth.log** sont des outils puissants pour surveiller l’intégrité de votre système et détecter les activités malveillantes. Associés à Syslog, ils permettent de centraliser, filtrer et analyser efficacement les événements critiques. En suivant ce guide, vous êtes prêt à identifier et à répondre aux incidents de sécurité. 🎉
