# Introduction à la commande `docker exec`

![Tutorial Cover](assets/docker-exec.jpg)

## Introduction

La commande `docker exec` permet **d’exécuter une commande dans un conteneur en cours d’exécution**. Cela est particulièrement utile pour **interagir avec un conteneur sans avoir à l’arrêter ou le redémarrer**, pour **déboguer une application**, ou encore pour **exécuter des tâches d’administration** à l’intérieur d’un conteneur.

Dans ce tutoriel, nous allons voir comment utiliser `docker exec` avec plusieurs cas pratiques, notamment :
- Exécuter des commandes simples.
- Accéder à un shell interactif.
- Vérifier et modifier un fichier dans un conteneur.
- Lancer un service ou un processus dans un conteneur.

---

## Partie 1 : Lancer un Conteneur pour les Tests

Avant de pouvoir utiliser `docker exec`, nous devons d'abord **avoir un conteneur en cours d’exécution**. 

Nous allons utiliser l’image officielle **Ubuntu** pour nos tests.

### Étape 1.1 : Démarrer un conteneur Ubuntu en arrière-plan

```bash
docker run -d --name ubuntu_test ubuntu sleep infinity
```

Explication :
- `-d` : Exécute le conteneur en arrière-plan.
- `--name ubuntu_test` : Donne un nom au conteneur (`ubuntu_test`).
- `ubuntu` : Utilise l’image officielle Ubuntu.
- `sleep infinity` : Empêche le conteneur de s’arrêter immédiatement.

### Étape 1.2 : Vérifier que le conteneur est bien en cours d’exécution

```bash
docker ps
```

!!! info
    Si tout est correct, vous devriez voir **ubuntu_test** en cours d’exécution.

---

## Partie 2 : Exécuter une Commande dans un Conteneur

Nous allons maintenant exécuter des commandes à l’intérieur du conteneur.

### Étape 2.1 : Vérifier la version d’Ubuntu

```bash
docker exec ubuntu_test cat /etc/os-release
```

Sortie attendue :
```bash
NAME="Ubuntu"
VERSION="..."
```

!!! info
    Cette commande **exécute `cat /etc/os-release`** dans le conteneur et affiche la version d’Ubuntu.

### Étape 2.2 : Lister les fichiers dans le conteneur

```bash
docker exec ubuntu_test ls -l /
```

Sortie attendue :
```bash
total ...
drwxr-xr-x   1 root root ... bin
drwxr-xr-x   1 root root ... boot
...
```

!!! info
    On peut voir la structure des fichiers **du système interne du conteneur**.

---

## Partie 3 : Accéder à un Shell Interactif

Si vous voulez **explorer un conteneur comme si vous étiez connecté directement à une machine**, utilisez un shell interactif.

### Étape 3.1 : Ouvrir un terminal dans le conteneur

```bash
docker exec -it ubuntu_test bash
```

Explication :
- `-i` : Active le mode interactif.
- `-t` : Associe un pseudo-terminal (tty).
- `bash` : Ouvre un shell **Bash** dans le conteneur.

!!! info
    Une fois la commande exécutée, votre invite de commande changera en quelque chose comme :
    ```bash
    root@<container_id>:/#
    ```
    Vous êtes maintenant **dans le conteneur** et pouvez exécuter des commandes comme sur une vraie machine Linux.

### Étape 3.2 : Quitter le conteneur

Pour quitter le shell, tapez :
```bash
exit
```
ou utilisez le raccourci `Ctrl + D`.

---

## Partie 4 : Vérifier et Modifier un Fichier dans un Conteneur

Nous allons maintenant **créer un fichier dans le conteneur** et vérifier qu’il est bien présent.

### Étape 4.1 : Créer un fichier dans le conteneur

Exécutez cette commande pour créer un fichier `/tmp/testfile.txt` à l’intérieur du conteneur :
```bash
docker exec ubuntu_test bash -c "echo 'Hello depuis le conteneur' > /tmp/testfile.txt"
```

### Étape 4.2 : Vérifier que le fichier existe

```bash
docker exec ubuntu_test cat /tmp/testfile.txt
```

Sortie attendue :
```bash
Hello depuis le conteneur
```

!!! info
    Cela prouve que nous avons bien **créé et modifié un fichier dans le conteneur**.

---

## Partie 5 : Lancer un Service dans un Conteneur

Nous allons maintenant tester le lancement d’un service **nginx** à l’intérieur d’un conteneur Ubuntu.

### Étape 5.1 : Installer nginx dans le conteneur

```bash
docker exec ubuntu_test apt update 
docker exec ubuntu_test apt install -y nginx
```

### Étape 5.2 : Démarrer nginx

```bash
docker exec ubuntu_test service nginx start
```

### Étape 5.3 : Vérifier que nginx fonctionne

```bash
docker exec ubuntu_test service nginx status
```

Sortie attendue :
```bash
* nginx is running
```

!!! info
    Nous avons installé et lancé un **serveur web Nginx dans le conteneur**.

---

## Exercice Pratique

### Objectif :
1. **Démarrer un conteneur basé sur Debian** (`debian:latest`).
2. **Créer un fichier `/root/exercice.txt`** dans le conteneur avec le texte **"Docker exec fonctionne !"**.
3. **Vérifier que le fichier a bien été créé**.
4. **Lancer un shell interactif dans le conteneur et afficher le contenu du fichier**.

---

## Conclusion

### Ce que nous avons appris :
- **Exécuter une commande simple dans un conteneur avec `docker exec`**.
- **Accéder à un shell interactif dans un conteneur**.
- **Créer, modifier et vérifier des fichiers à l’intérieur d’un conteneur**.
- **Lancer un service (ex. Nginx) à l’intérieur d’un conteneur**.

`docker exec` est une commande **puissante** qui vous permet d’interagir avec vos conteneurs **sans les arrêter**, idéale pour le débogage et l’administration système.
