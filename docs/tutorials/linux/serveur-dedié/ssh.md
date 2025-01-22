
# Linus - Configuration de ssh

![Tutorial Cover](assets/ssh.jpg)

## Introduction

Ce guide vous accompagne pas à pas dans la configuration de SSH sur un serveur Linux et dans sa validation en local (via `localhost`).

---

## Pré-requis
- Une machine Linux (ex. Ubuntu, Debian, SUSE, CentOS).
- Les privilèges root ou accès sudo.
- OpenSSH installé (client et serveur).

---

## Étape 1 : Vérifier si SSH est installé

Exécutez la commande suivante pour vérifier si SSH est installé :

```bash
sudo systemctl status ssh
```

- Si SSH est installé, vous verrez un statut indiquant `active (running)`.
- Si SSH n'est pas installé, installez-le avec :
  ```bash
  # Pour les distributions basées sur Debian/Ubuntu
  sudo apt update && sudo apt install openssh-server

  # Pour les distributions basées sur RedHat/CentOS
  sudo yum install openssh-server

  # Pour les distributions basées sur SUSE
  sudo zypper install openssh
  ```

---

## Étape 2 : Configuration du Serveur SSH

Le fichier de configuration principal de SSH se trouve dans `/etc/ssh/sshd_config`. 

1. **Ouvrir le fichier de configuration** :
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. **Vérifiez et modifiez les paramètres essentiels** :
   - **Changer le port SSH (optionnel)** :  
     Remplacez `#Port 22` par un autre numéro, par exemple `Port 2222`. Cela ajoute une couche de sécurité.
   - **Autoriser uniquement SSH v2** (plus sécurisé) :
     Assurez-vous que `Protocol 2` est activé.
   - **Restreindre les connexions root** :
     Remplacez `PermitRootLogin yes` par `PermitRootLogin no` pour empêcher les connexions root.
   - **Limiter les utilisateurs autorisés** (optionnel) :
     Ajoutez `AllowUsers <utilisateur>` pour restreindre l'accès à certains utilisateurs.

3. **Sauvegardez et fermez le fichier**.

---

## Étape 3 : Redémarrer le Service SSH

Pour appliquer les modifications, redémarrez le service SSH :

```bash
sudo systemctl restart ssh
```

---

## Étape 4 : Tester la Configuration SSH en Local

Pour tester la configuration, nous utiliserons `localhost`.

1. **Vérifier le port d'écoute** :
   ```bash
   sudo netstat -tuln | grep ssh
   ```
   !!! info
   La commande ci-dessus vous montre si SSH écoute sur le port configuré (par défaut `22` ou celui défini).

2. **Tester une connexion SSH locale** :
   ```bash
   ssh <votre_utilisateur>@localhost
   ```
   Remplacez `<votre_utilisateur>` par votre nom d'utilisateur. Vous devriez être invité à entrer un mot de passe.

3. **Tester un autre port (si configuré)** :
   ```bash
   ssh -p 2222 <votre_utilisateur>@localhost
   ```
   !!! info
   Si vous avez changé le port dans `/etc/ssh/sshd_config`, cette commande vérifie que la connexion fonctionne correctement.

4. **Vérification des logs** :
   Si un problème survient, consultez les logs du service SSH :
   ```bash
   sudo journalctl -u ssh
   ```

---

## Étape 5 : Configuration des Clés SSH (Optionnel mais Recommandé)

1. **Générer une paire de clés SSH** :
   Exécutez la commande suivante en tant qu'utilisateur :
   ```bash
   ssh-keygen -t rsa -b 4096
   ```
   Suivez les instructions pour sauvegarder la clé dans le fichier par défaut (`~/.ssh/id_rsa`).

2. **Copier la clé publique sur le serveur (local)** :
   ```bash
   ssh-copy-id -i ~/.ssh/id_rsa.pub <votre_utilisateur>@localhost
   ```
   Cela ajoute votre clé publique au fichier `~/.ssh/authorized_keys`.

3. **Tester la connexion par clé SSH** :
   ```bash
   ssh <votre_utilisateur>@localhost
   ```
   !!! info
   Si configuré correctement, la connexion devrait s'établir sans demander de mot de passe.

---

## Étape 6 : Désactiver l'Authentification par Mot de Passe (Optionnel)

Pour renforcer la sécurité, désactivez l'authentification par mot de passe après avoir configuré les clés SSH.

1. **Modifier le fichier de configuration** :
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```
   Recherchez et modifiez la ligne suivante :
   ```text
   PasswordAuthentication no
   ```

2. **Redémarrer SSH** :
   ```bash
   sudo systemctl restart ssh
   ```

3. **Tester la connexion** :
   ```bash
   ssh <votre_utilisateur>@localhost
   ```
   !!! info
   Si tout est configuré correctement, seul l'accès par clé SSH sera autorisé.

---

## Étape 7 : Sécuriser davantage le Serveur SSH

- **Activer le pare-feu** :
  ```bash
  sudo ufw allow 22/tcp
  sudo ufw enable
  ```

- **Limiter les tentatives de connexion** (exemple avec fail2ban) :
  ```bash
  sudo apt install fail2ban
  sudo systemctl enable fail2ban
  ```

---

## Étape 8 : Résolution des Problèmes Courants

1. **Vérifier si SSH est actif** :
   ```bash
   sudo systemctl status ssh
   ```

2. **Consulter les logs** :
   ```bash
   sudo journalctl -u ssh
   ```

3. **Vérifier les permissions des fichiers** :
   ```bash
   ls -ld ~/.ssh ~/.ssh/authorized_keys
   ```
   Assurez-vous que :
   - `.ssh` a des permissions `700`.
   - `authorized_keys` a des permissions `600`.

---