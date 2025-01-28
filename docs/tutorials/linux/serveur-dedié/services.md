# Linus - Sécurisation des Services avec Fail2ban et PortSentry

![Tutorial Cover](assets/security.jpg)

## Introduction

La sécurité des serveurs est essentielle dans un environnement connecté. **Fail2ban** et **PortSentry** sont deux outils complémentaires permettant de renforcer la défense contre les attaques courantes. Ce guide explique leur rôle, leur fonctionnement et leur configuration.

---

## Pourquoi utiliser Fail2ban et PortSentry ?

### Fail2ban
Fail2ban surveille les journaux de votre serveur pour détecter des comportements suspects comme des tentatives répétées de connexion (attaques par force brute). Lorsqu'une activité malveillante est détectée, Fail2ban bloque temporairement l'IP fautive en configurant automatiquement le pare-feu.

#### **Avantages de Fail2ban** :
- Protège les services spécifiques (ex. SSH, HTTP, FTP).
- Réduit les risques d'accès non autorisés par force brute.
- Flexible grâce à ses filtres personnalisables.

---

### PortSentry
PortSentry est conçu pour détecter les scans de ports, une technique utilisée par les attaquants pour identifier les services actifs sur un serveur. Lorsqu'un scan est détecté, PortSentry bloque immédiatement l'IP suspecte via le pare-feu ou d'autres méthodes.

#### **Avantages de PortSentry** :
- Détecte et bloque les attaques en amont.
- Complète Fail2ban en sécurisant les services "non exposés".
- Agit en mode préventif contre les tentatives de reconnaissance.

---

## Partie 1 : Installation et Configuration de Fail2ban

### Étape 1.1 : Installation de Fail2ban

1. **Installer Fail2ban** :  
   Utilisez le gestionnaire de paquets de votre distribution :
   ```bash
   # Ubuntu/Debian
   sudo apt update && sudo apt install fail2ban

   # CentOS/RedHat
   sudo yum install fail2ban

   # SUSE
   sudo zypper install fail2ban
   ```

2. **Vérifiez que Fail2ban est installé et actif** :
   ```bash
   sudo systemctl status fail2ban
   ```

---

### Étape 1.2 : Configurer Fail2ban

Fail2ban utilise des **jails** (prisons) pour définir des règles spécifiques à chaque service.

1. **Créer un fichier de configuration local** :
   ```bash
   sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
   ```

2. **Configurer les options générales** :
   - Ouvrez le fichier `/etc/fail2ban/jail.local` :
     ```bash
     sudo nano /etc/fail2ban/jail.local
     ```
   - Modifiez les paramètres principaux :
     ```text
     ignoreip = 127.0.0.1/8 ::1  # Exclure localhost des bannissements
     bantime = 3600             # Durée du bannissement (en secondes)
     maxretry = 3               # Nombre de tentatives avant le bannissement
     ```

3. **Activer la protection pour SSH** :
   Recherchez la section `[sshd]` et configurez-la ainsi :
   ```text
   [sshd]
   enabled = true
   port = ssh
   logpath = /var/log/auth.log
   maxretry = 3
   ```

---

### Étape 1.3 : Tester Fail2ban

1. **Redémarrez Fail2ban pour appliquer les modifications** :
   ```bash
   sudo systemctl restart fail2ban
   ```

2. **Simulez une attaque par force brute sur SSH** :  
   - Essayez de vous connecter avec un mot de passe incorrect :
     ```bash
     ssh invalid_user@localhost
     ```
   - Après plusieurs tentatives échouées (selon `maxretry`), votre IP sera bannie.

3. **Vérifiez les IP bannies** :
   ```bash
   sudo fail2ban-client status sshd
   ```

4. **Débannir une IP (si nécessaire)** :
   ```bash
   sudo fail2ban-client unban <IP>
   ```

!!! info
    Si vous vous bannissez par erreur, utilisez une autre session pour débannir votre IP.

---

## Partie 2 : Installation et Configuration de PortSentry

### Étape 2.1 : Installation de PortSentry

1. **Installer PortSentry** :
   ```bash
   # Ubuntu/Debian
   sudo apt update && sudo apt install portsentry

   # CentOS/RedHat (via EPEL)
   sudo yum install portsentry

   # SUSE
   sudo zypper install portsentry
   ```

2. **Vérifiez l’installation** :
   ```bash
   portsentry -version
   ```

---

### Étape 2.2 : Configurer PortSentry

1. **Modifier le fichier de configuration principal** :  
   ```bash
   sudo nano /etc/portsentry/portsentry.conf
   ```

2. **Configurer les ports surveillés** :  
   Modifiez ou ajoutez une liste personnalisée de ports :
   ```text
   TCP_PORTS="1,11,15,23,79,81,443,1080"
   UDP_PORTS="1,7,9,69,135,137,161,500"
   ```

3. **Activer le mode Stealth** :  
   Le mode Stealth détecte les scans furtifs et réagit en bloquant l'IP :
   ```text
   BLOCK_UDP="1"
   BLOCK_TCP="1"
   ```

4. **Configurer les actions en cas de détection** :  
   Pour bloquer une IP via iptables :
   ```text
   KILL_ROUTE="/sbin/iptables -I INPUT -s $TARGET$ -j DROP"
   ```

---

### Étape 2.3 : Tester PortSentry

1. **Démarrez PortSentry** :
   ```bash
   sudo systemctl start portsentry
   ```

2. **Simulez une tentative de scan de port** :  
   Utilisez `nmap` pour scanner les ports :
   ```bash
   nmap localhost
   ```
   - Si PortSentry est actif, il détectera le scan et bloquera l'IP.

3. **Vérifiez les logs** :
   ```bash
   sudo tail -f /var/log/syslog
   ```
   Les actions de PortSentry y sont enregistrées.

4. **Vérifiez les règles iptables ajoutées** :
   ```bash
   sudo iptables -L -v
   ```

---

### Étape 2.4 : Débannir une IP dans PortSentry

Lorsque PortSentry bannit une IP, elle est ajoutée aux règles iptables ou au fichier `hosts.deny`.

1. **Débannir une IP dans iptables** :
   ```bash
   sudo iptables -D INPUT -s <IP_BANNEE> -j DROP
   ```

2. **Supprimer une IP bannie dans `/etc/hosts.deny`** (si configuré) :
   - Ouvrez le fichier :
     ```bash
     sudo nano /etc/hosts.deny
     ```
   - Recherchez et supprimez la ligne contenant l'IP bannie.

3. **Redémarrez PortSentry** :
   ```bash
   sudo systemctl restart portsentry
   ```

!!! info
    Débannir manuellement une IP peut être nécessaire si vous bloquez accidentellement une adresse autorisée, comme localhost.

---

## Conclusion

Fail2ban et PortSentry permettent de sécuriser efficacement vos services contre les attaques courantes et les tentatives d’intrusion. En combinant ces outils, vous renforcez significativement la sécurité de votre serveur. Pensez à tester régulièrement vos règles sur `localhost` avant une mise en production. 🎉
