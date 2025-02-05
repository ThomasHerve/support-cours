# Linus - Introduction à l'option `-v` de Docker (Volumes)

![Tutorial Cover](assets/docker-volumes.jpg)

## Introduction

L’option `-v` (ou `--volume`) de Docker permet de **monter des volumes**, c'est-à-dire de lier un dossier du système hôte à un dossier à l’intérieur d’un conteneur. Cette fonctionnalité est essentielle pour **partager des fichiers** entre l’hôte et le conteneur, **préserver des données** même après l’arrêt d’un conteneur et **modifier le contenu du conteneur en temps réel**.

Dans ce tutoriel, nous allons voir comment utiliser `-v` pour monter un volume et modifier dynamiquement les fichiers d’un serveur **Nginx** en conteneur.

---

## Partie 1 : Lancer un Conteneur Nginx Normal

### Étape 1.1 : Télécharger l’image Nginx 

Avant de lancer un conteneur Nginx, assurez-vous d’avoir l’image officielle :
```bash
docker pull nginx
```

### Étape 1.2 : Lancer un conteneur Nginx simple

Exécutez la commande suivante pour démarrer un conteneur Nginx sans volume :
```bash
docker run -p 8080:80 --name nginx_no_volume nginx
```

### Étape 1.3 : Vérifier l’accès au serveur

!!! info
    Ouvrez votre navigateur et allez à **http://localhost:8080**. Vous devriez voir la **page par défaut de Nginx**.

---

## Partie 2 : Remplacer le HTML par un Contenu Personnalisé avec `-v`

Nous allons maintenant utiliser un volume pour **remplacer la page par défaut de Nginx** par une page HTML personnalisée.

### Étape 2.1 : Créer un Dossier et un Fichier HTML

Sur votre ordinateur, créez un dossier pour contenir notre page HTML personnalisée:
```bash
mkdir -p <CHEMIN>/nginx_html
```

Créez un fichier `index.html` à l’intérieur :
```bash
echo "<h1>Bienvenue sur mon site personnalisé !</h1>" > ~/docker/nginx_html/index.html
```

### Étape 2.2 : Lancer Nginx avec un Volume

Arrêtez et supprimez l'ancien conteneur :
```bash
docker stop nginx_no_volume  
docker rm nginx_no_volume
```

Lancez un nouveau conteneur Nginx en **montant notre dossier** en tant que volume :
```bash
docker run -d -p 8080:80 --name nginx_with_volume -v <CHEMIN>/nginx_html:/usr/share/nginx/html nginx
```

Explication :
- `-v <CHEMIN>/nginx_html:/usr/share/nginx/html` : Monte notre dossier local (`<CHEMIN>/nginx_html`) à la place du dossier par défaut de Nginx (`/usr/share/nginx/html`).

### Étape 2.3 : Vérifier que la page a changé

!!! info
    Retournez sur **http://localhost:8080** et actualisez la page. Vous devriez voir **"Bienvenue sur mon site personnalisé !"** au lieu de la page par défaut de Nginx.

Vous pouvez également tester avec :
```bash
curl http://localhost:8080
```
Vous devriez voir en sortie :
```html
<h1>Bienvenue sur mon site personnalisé !</h1>
```

---

## Partie 3 : Modification Dynamique du HTML

Une des forces des volumes est que les modifications sur l’hôte sont immédiatement prises en compte par le conteneur.

### Étape 3.1 : Modifier le fichier HTML

Éditez le fichier `index.html` :
```bash
echo "<h1>Page mise à jour avec succès !</h1>" > ~/docker/nginx_html/index.html
```

### Étape 3.2 : Vérifier le changement

!!! info
    Actualisez **http://localhost:8080**. La modification est immédiatement visible, sans redémarrer le conteneur.

Vous devriez voir :
```html
<h1>Page mise à jour avec succès !</h1>
```

---

## Exercice Pratique

### Objectif :
1. **Créer un nouveau conteneur Nginx** nommé `nginx_test_volume`.
2. **Créer un volume** contenant un fichier HTML (`index.html`) avec le texte **"Exercice réussi !"**.
3. **Monter le volume** dans le conteneur pour qu'il affiche ce fichier HTML.
4. **Vérifier que la page affiche bien "Exercice réussi !"**.
---

## Conclusion

### Ce que nous avons appris :
- **Comment utiliser `-v` pour monter un volume** et modifier dynamiquement les fichiers d’un conteneur.
- **L'impact des volumes** sur la persistance des données dans Docker.
- **Comment modifier le contenu d’un conteneur sans le redémarrer**.

L’option `-v` est **indispensable** pour la gestion des fichiers dans Docker, que ce soit pour la configuration de services web, la gestion de bases de données ou le stockage de logs.
