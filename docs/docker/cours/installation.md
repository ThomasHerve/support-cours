#  Introduction à l'Installation de Docker sur Linux et Windows

![Tutorial Cover](assets/docker-installation.jpg)

## Introduction

Docker est une plateforme de virtualisation légère qui permet de déployer des applications dans des conteneurs. Ces conteneurs sont des environnements isolés, offrant une grande portabilité et permettant d'éviter les problèmes liés aux environnements de développement et de production. Dans ce tutoriel, nous allons couvrir l'installation de Docker sur **Linux** et **Windows**, et vous fournir les étapes nécessaires pour commencer à travailler avec Docker dans ces deux environnements.

---

## Partie 1 : Installation de Docker sur Linux

Docker est très bien supporté sur la plupart des distributions Linux. Nous allons couvrir l'installation sur une distribution basée sur Debian/Ubuntu et RedHat/CentOS.

### Étape 1.1 : Installation de Docker sur Ubuntu/Debian

1. **Mettre à jour le système** :
   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

2. **Installer les dépendances nécessaires** :
   Docker nécessite certains paquets pour fonctionner correctement, installez-les avec :
   ```bash
   sudo apt install apt-transport-https ca-certificates curl software-properties-common
   ```

3. **Ajouter la clé GPG officielle de Docker** :
   Exécutez la commande suivante pour ajouter la clé GPG officielle de Docker à votre système :
   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```

4. **Ajouter le dépôt Docker** :
   Ajoutez le dépôt Docker aux sources de votre système pour pouvoir installer Docker :
   ```bash
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   ```

5. **Mettre à jour les sources et installer Docker** :
   ```bash
   sudo apt update
   sudo apt install docker-ce docker-ce-cli containerd.io
   ```

6. **Vérifier l'installation** :
   Pour vérifier que Docker est correctement installé, exécutez :
   ```bash
   sudo docker --version
   ```

7. **Démarrer et activer Docker** :
   Si Docker n’est pas déjà démarré, lancez-le et assurez-vous qu'il se lance automatiquement au démarrage :
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

8. **Vérifier que Docker fonctionne** :
   Vous pouvez vérifier que Docker fonctionne correctement avec la commande suivante :
   ```bash
   sudo docker run hello-world
   ```

   Cette commande téléchargera une image de test et exécutera un conteneur, affichant un message de bienvenue si tout est bien configuré.

---

### Étape 1.2 : Installation de Docker sur CentOS/RHEL

1. **Mettre à jour le système** :
   ```bash
   sudo yum update -y
   ```

2. **Installer les dépendances** :
   ```bash
   sudo yum install -y yum-utils device-mapper-persistent-data lvm2
   ```

3. **Ajouter le dépôt Docker** :
   Ajoutez le dépôt Docker officiel à vos sources :
   ```bash
   sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
   ```

4. **Installer Docker** :
   Installez Docker avec :
   ```bash
   sudo yum install docker-ce docker-ce-cli containerd.io
   ```

5. **Démarrer et activer Docker** :
   Lancez Docker et assurez-vous qu'il se lance au démarrage :
   ```bash
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

6. **Vérifier l'installation** :
   Vérifiez que Docker est bien installé :
   ```bash
   sudo docker --version
   ```

7. **Exécuter un test** :
   Comme sur Ubuntu/Debian, vous pouvez tester l'installation avec :
   ```bash
   sudo docker run hello-world
   ```

---

## Partie 2 : Installation de Docker sur Windows avec Docker Desktop

### Étape 2.1 : Télécharger Docker Desktop

1. Rendez-vous sur la page officielle de téléchargement de Docker Desktop pour Windows :
   [Docker Desktop - Télécharger](https://www.docker.com/products/docker-desktop)

2. Cliquez sur **Download for Windows** pour télécharger l'installateur.

3. Une fois le fichier téléchargé, lancez l'installateur.

---

### Étape 2.2 : Installation de Docker Desktop

1. **Lancer l'installateur** :
   Ouvrez le fichier téléchargé et suivez les étapes de l'installation. L'installateur va vous guider à travers les étapes suivantes :
   - Accepter les conditions d'utilisation.
   - Installer les composants nécessaires (Docker Desktop, WSL 2 si nécessaire).
   - Demander un redémarrage du système à la fin de l'installation.

2. **Activer WSL 2** :
   Docker Desktop pour Windows utilise **Windows Subsystem for Linux (WSL) 2**. Si WSL 2 n’est pas déjà installé, Docker vous guidera pour l’installer pendant l'installation.

   Si nécessaire, suivez ces étapes pour activer WSL 2 :
   - Installez WSL via PowerShell en tant qu'administrateur :
     ```powershell
     wsl --install
     ```
   - Redémarrez votre PC après l’installation de WSL.

---

### Étape 2.3 : Lancer Docker Desktop

1. Une fois l'installation terminée, ouvrez **Docker Desktop** en cliquant sur l'icône correspondante dans le menu démarrer ou la barre des tâches.

2. Docker va se lancer et vous pourrez accéder à l'interface graphique de Docker.

---

### Étape 2.4 : Vérifier l'installation

1. Ouvrez une fenêtre PowerShell ou **CMD** et tapez la commande suivante pour vérifier que Docker fonctionne :
   ```powershell
   docker --version
   ```

2. Lancez un conteneur de test pour vérifier que tout fonctionne bien :
   ```powershell
   docker run hello-world
   ```

   Vous devriez voir un message de bienvenue qui confirme que Docker fonctionne correctement.

---

## Partie 3 : Conclusion

### Résumé de l'installation

- **Sur Linux** : L'installation de Docker est relativement simple avec les gestionnaires de paquets comme `apt` pour Ubuntu/Debian ou `yum` pour CentOS/RHEL. Après l'installation, Docker peut être testé avec la commande `docker run hello-world`.
  
- **Sur Windows** : Docker Desktop est l'outil recommandé pour utiliser Docker sur Windows. Il s’appuie sur WSL 2 pour offrir une expérience proche de celle d’un environnement Linux natif.

### Résultats attendus

- Sur **Linux**, vous serez en mesure de lancer des conteneurs en utilisant Docker.
- Sur **Windows**, Docker Desktop vous permettra d'exécuter des conteneurs et de gérer vos images Docker grâce à une interface graphique simple et des commandes en ligne via PowerShell.

Félicitations ! Vous avez maintenant Docker installé et vous pouvez commencer à explorer le monde des conteneurs !
