# Linux - Analyse des Logs de Sécurité avec Auth.log

![Tutorial Cover](assets/authlog-syslog.jpg)

## Introduction

Les journaux de sécurité sont une ressource précieuse pour détecter des connexions suspectes, des tentatives de brute-force ou des activités non autorisées. Ce guide vous explique comment utiliser **auth.log** (le fichier journal dédié aux authentifications).

---

## Pourquoi analyser auth.log ?

### Auth.log
- **Suivi des authentifications** : Captures des connexions SSH, sudo, échecs de connexion, etc.
- **Détection d'activités suspectes** : Identifier des IP malveillantes ou des tentatives d'accès non autorisées.
- **Dépannage** : Aide à résoudre les problèmes liés aux permissions et aux authentifications.
---

## Partie 1 : Configuration et Accès aux Logs

### Étape 1.1: Setup docker 

Comme dans le tp précédent, nous allons utiliser docker pour pouvoir utiliser des outils qui autrement demanderais un linux natif.

1. Créez un nouveau dossier de travail
2. Copiez votre clé id_rsa.pub dans votre dossier de travail
3. Créez le dockerfile
```dockerfile
FROM ubuntu:latest

RUN apt-get update && apt-get install -y openssh-server

RUN useradd -m -s /bin/bash test&& echo "test:123456" | chpasswd
RUN mkdir -p /home/test/.ssh 
RUN chown test:test /home/test/.ssh 
RUN chmod 700 /home/test/.ssh

RUN mkdir -p /etc/ssh/logs && chown root:root /etc/ssh/logs && chmod 700 /etc/ssh/logs

RUN mkdir -p /run/sshd && chown root:root /run/sshd && chmod 700 /run/sshd

COPY id_rsa.pub /home/test/.ssh/authorized_keys
RUN chmod 644 /home/test/.ssh/authorized_keys

RUN sed -i '1i PasswordAuthentication no' /etc/ssh/sshd_config
RUN sed -i '1i PermitEmptyPasswords no' /etc/ssh/sshd_config
RUN sed -i '1i KbdInteractiveAuthentication no' /etc/ssh/sshd_config
RUN sed -i '1i UsePAM yes' /etc/ssh/sshd_config
RUN sed -i '1i PubkeyAuthentication yes' /etc/ssh/sshd_config
RUN sed -i '1i X11Forwarding no' /etc/ssh/sshd_config
RUN sed -i '1i PermitRootLogin no' /etc/ssh/sshd_config
RUN sed -i '1i StrictModes yes' /etc/ssh/sshd_config
RUN sed -i '1i LogLevel INFO' /etc/ssh/sshd_config

RUN chmod 600 /etc/ssh/sshd_config

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D", "-E", "/etc/ssh/logs/auth.log"]
```

4. Créez un fichier docker-compose.yml
```yaml
services:
  ssh:
    build: .
    container_name: ssh
    ports:
      - 2222:22

    volumes:
      - ./logs:/etc/ssh/logs
```


### Étape 1.2: Démarrer l'env

1. Démarrer l’environnement Docker :
   ```bash
   docker-compose up -d
   ```

2. Vérifier que les conteneurs tournent :
   ```text
   docker ps
   ```

### Étape 1.3: Tester

1. Connectez vous avec :
   ```bash
   ssh test@localhost -p 2222
   ```
2. Vous pouvez directement consulter le fichier depuis votre systeme de fichier dans logs/auth.log à coté de cotre docker-compose.yaml

## Partie 2 : Analyse des Logs avec Auth.log

### Étape 2.1 : Détection de Connexions SSH

1. Points à analyser :
   \- **Connexions réussies** :
     ```text
     Accepted password for <user> from <IP> port <port> ssh2
     ```
   \- **Tentatives échouées** :
     ```text
     Failed password for invalid user <user> from <IP> port <port> ssh2
     ```
   \- Essayez de faire apparaitre ces lignes
