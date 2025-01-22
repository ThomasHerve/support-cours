# Linus - Configuration des Pare-feu : UFW et iptables

![Tutorial Cover](assets/firewall.jpg)

## Introduction

Les pare-feu sont des outils essentiels pour sécuriser un serveur Linux. Ce guide explique en détail la configuration des deux principaux pare-feu : **UFW** (Uncomplicated Firewall) et **iptables**. Nous explorerons leurs fonctionnalités, leur configuration et leur validation.

---

## Pré-requis
- Une machine Linux avec un accès root ou sudo.
- Les outils **UFW** et **iptables** installés.
- Une compréhension de base des ports et des protocoles réseau.

---

## Partie 1 : Utilisation de UFW

**UFW** est une interface simplifiée pour gérer iptables, conçue pour les administrateurs système novices.

### Étape 1.1 : Vérifier si UFW est installé

Exécutez la commande suivante pour vérifier si UFW est installé :

```bash
sudo ufw status
```

- Si UFW est installé, vous verrez un statut (par exemple `active` ou `inactive`).
- Sinon, installez-le :
  ```bash
  sudo apt update && sudo apt install ufw
  ```

!!! info
    Une fois installé, assurez-vous qu'aucune règle par défaut ne bloque `localhost`.

---

### Étape 1.2 : Configurer les règles de base

1. **Autoriser les connexions SSH (indispensable)** :
   ```bash
   sudo ufw allow ssh
   ```

2. **Autoriser ou bloquer des ports spécifiques** :
   - Autoriser le HTTP (port 80) :
     ```bash
     sudo ufw allow 80/tcp
     ```
   - Autoriser le HTTPS (port 443) :
     ```bash
     sudo ufw allow 443/tcp
     ```
   - Bloquer un port :
     ```bash
     sudo ufw deny 21/tcp
     ```

3. **Autoriser un IP ou un réseau spécifique** :
   - Autoriser une IP à se connecter à tout le serveur :
     ```bash
     sudo ufw allow from 192.168.1.100
     ```
   - Autoriser une IP pour un port spécifique :
     ```bash
     sudo ufw allow from 192.168.1.100 to any port 22
     ```

---

### Étape 1.3 : Activer UFW

Activez UFW pour que les règles prennent effet :

```bash
sudo ufw enable
```

!!! info
    Une fois activé, toutes les connexions non autorisées seront bloquées par défaut.

---

### Étape 1.4 : Tester la configuration UFW en local

1. **Vérifiez les ports autorisés** :
   Utilisez `nmap` pour scanner les ports de `localhost` :
   ```bash
   sudo apt install nmap
   nmap localhost
   ```

2. **Tester les règles spécifiques** :
   - Pour tester si le port HTTP (80) est ouvert :
     ```bash
     curl http://localhost
     ```
   - Pour tester un port bloqué (par exemple FTP sur 21) :
     ```bash
     nc -zv localhost 21
     ```

3. **Vérifiez les logs UFW** :
   ```bash
   sudo tail -f /var/log/ufw.log
   ```

---

## Partie 2 : Utilisation de iptables

**iptables** offre un contrôle plus fin sur le filtrage des paquets réseau. Cependant, il est plus complexe à utiliser.

### Étape 2.1 : Vérifier si iptables est installé

Pour vérifier si iptables est disponible sur votre système :

```bash
sudo iptables -L
```

Si la commande retourne des chaînes de règles (INPUT, FORWARD, OUTPUT), alors iptables est actif.

---

### Étape 2.2 : Configurer des règles simples

1. **Bloquer tout le trafic entrant par défaut** :
   ```bash
   sudo iptables -P INPUT DROP
   ```

2. **Autoriser le trafic sortant** :
   ```bash
   sudo iptables -P OUTPUT ACCEPT
   ```

3. **Autoriser uniquement SSH** :
   ```bash
   sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
   ```

4. **Autoriser un réseau spécifique** :
   ```bash
   sudo iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT
   ```

---

### Étape 2.3 : Sauvegarder et restaurer les règles

Les modifications d’iptables ne sont pas persistantes par défaut. Pour sauvegarder vos règles :

1. **Sauvegarder les règles actuelles** :
   ```bash
   sudo iptables-save > /etc/iptables.rules
   ```

2. **Restaurer les règles au démarrage** :
   Ajoutez cette ligne dans `/etc/rc.local` :
   ```bash
   iptables-restore < /etc/iptables.rules
   ```

---

### Étape 2.4 : Tester la configuration iptables en local

1. **Vérifiez les règles appliquées** :
   ```bash
   sudo iptables -L -v
   ```

2. **Scanner les ports ouverts** :
   ```bash
   nmap localhost
   ```

3. **Tester un port spécifique** :
   - Pour vérifier si le port SSH (22) est accessible :
     ```bash
     nc -zv localhost 22
     ```
   - Pour un port bloqué, comme le port HTTP si non autorisé :
     ```bash
     curl http://localhost
     ```

4. **Vérifiez les logs d’iptables** :
   Ajoutez une règle pour journaliser les paquets rejetés :
   ```bash
   sudo iptables -A INPUT -j LOG --log-prefix "IPTables-Dropped: "
   sudo tail -f /var/log/kern.log
   ```

---

## Conclusion

Vous avez maintenant une configuration fonctionnelle des pare-feu UFW et iptables. Testez toujours vos règles sur `localhost` avant de les appliquer à un environnement en production pour éviter de vous bloquer hors de votre système. 🎉
