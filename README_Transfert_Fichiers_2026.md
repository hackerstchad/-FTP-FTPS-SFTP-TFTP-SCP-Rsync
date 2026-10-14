# Guide Avancé des Protocoles de Transfert de Fichiers

**Auteur :** `hackers_tchad`  
**Date de publication :** 2026  
**Version :** 2.0 — Référence complète FTP / FTPS / SFTP / TFTP / SCP / Rsync  
**Licence :** Document éducatif et référentiel à vocation défensive.  
**Contact / communauté :** hackers_tchad — veille cybersécurité & administration système.

---

# Table des matières

1. [Introduction générale](#introduction-générale)
2. [Architecture réseau et modèles de référence](#architecture-réseau-et-modèles-de-référence)
3. [Comparatif rapide des protocoles](#comparatif-rapide-des-protocoles)
4. [FTP — File Transfer Protocol](#ftp--file-transfer-protocol)
   - 3.1 Description et historique
   - 3.2 Qui a créé FTP ?
   - 3.3 Où et quand utiliser FTP ?
   - 3.4 Architecture et modes de fonctionnement
   - 3.5 Ports utilisés
   - 3.6 Installation et configuration
   - 3.7 Commandes FTP essentielles
   - 3.8 Vérification / dépannage sous Linux et Windows
   - 3.9 Envoi et réception de fichiers
   - 3.10 Sécurité et bonnes pratiques
   - 3.11 Outils clients FTP
   - 3.12 Ressources et livres
4. [FTPS — FTP over SSL/TLS](#ftps--ftp-over-ssltls)
   - 4.1 Description et historique
   - 4.2 Différences FTP vs FTPS
   - 4.3 Modes FTPS : explicite et implicite
   - 4.4 Configuration serveur FTPS
   - 4.5 Vérification sous Linux et Windows
   - 4.6 Commandes et opérations
   - 4.7 Bonnes pratiques
5. [SFTP — SSH File Transfer Protocol](#sftp--ssh-file-transfer-protocol)
   - 5.1 Description et historique
   - 5.2 Différences SFTP vs FTPS vs SCP
   - 5.3 Installation et configuration
   - 5.4 Authentification par clé SSH
   - 5.5 Commandes SFTP
   - 5.6 Vérification sous Linux et Windows
   - 5.7 Scripts et automatisation
   - 5.8 Sécurité
6. [TFTP — Trivial File Transfer Protocol](#tftp--trivial-file-transfer-protocol)
   - 6.1 Description et historique
   - 6.2 Cas d'usage
   - 6.3 Installation et configuration
   - 6.4 Commandes TFTP
   - 6.5 Vérification sous Linux et Windows
   - 6.6 Limites et sécurité
7. [SCP — Secure Copy Protocol](#scp--secure-copy-protocol)
   - 7.1 Description et historique
   - 7.2 Syntaxe SCP
   - 7.3 Exemples avancés
   - 7.4 Vérification sous Linux et Windows
   - 7.5 Remplacement par SFTP / Rsync
8. [Rsync](#rsync)
   - 8.1 Description et historique
   - 8.2 Algorithmes et optimisation
   - 8.3 Installation
   - 8.4 Syntaxe et options
   - 8.5 Synchronisation locale et distante
   - 8.6 Intégration SSH
   - 8.7 Vérification sous Linux et Windows
   - 8.8 Sauvegardes et rotations
9. [Synthèse des commandes de vérification](#synthèse-des-commandes-de-vérification)
10. [Applications et outils graphiques](#applications-et-outils-graphiques)
11. [Livres, ressources et liens](#livres-ressources-et-liens)
12. [Glossaire](#glossaire)

---

# Introduction générale

Le transfert de fichiers est une des fonctions les plus anciennes et les plus fondamentales des réseaux informatiques. Dès les années 1970, les ingénieurs ont conçu des protocoles permettant d'envoyer des documents, des programmes ou des données d'une machine à une autre via un réseau. Aujourd'hui, ces protocoles sont omniprésents : du simple téléchargement d'une page web à la sauvegarde automatisée de serveurs, du déploiement de firmware sur un routeur à l'échange sécurisé de documents sensibles entre institutions.

Ce guide présente six protocoles majeurs de transfert de fichiers :

- **FTP** (File Transfer Protocol) : le plus ancien, fondamental mais non sécurisé.
- **FTPS** (FTP over SSL/TLS) : version sécurisée de FTP utilisant TLS/SSL.
- **SFTP** (SSH File Transfer Protocol) : protocole de transfert sur tunnel SSH.
- **TFTP** (Trivial File Transfer Protocol) : version minimale sans authentification.
- **SCP** (Secure Copy Protocol) : copie simple et sécurisée via SSH.
- **Rsync** : outil de synchronisation efficace et polyvalent, souvent sur SSH.

Chaque section décrit l'historique, les cas d'usage, l'architecture, les ports, l'installation, la configuration, les commandes de vérification sous Linux et Windows, les opérations pratiques, les outils associés, ainsi que des ressources complémentaires (livres, liens, applications).

---

# Architecture réseau et modèles de référence

## Modèle OSI

Le modèle OSI (Open Systems Interconnection) définit sept couches pour décrire les communications réseau. Les protocoles de transfert de fichiers s'inscrivent principalement dans les couches application (7), présentation (6) et session (5), en s'appuyant sur les couches transport (4) et réseau (3).

| Couche OSI | Rôle | Protocoles associés |
|------------|------|---------------------|
| 7 — Application | Interface utilisateur et services réseau | FTP, FTPS, SFTP, TFTP, SCP, Rsync |
| 6 — Présentation | Encodage, chiffrement, compression | TLS/SSL, SSH, ASCII/binary FTP |
| 5 — Session | Gestion des sessions | Connexions FTP/SFTP persistentes |
| 4 — Transport | Fiabilité et ports | TCP (FTP, FTPS, SFTP, SCP, Rsync), UDP (TFTP) |
| 3 — Réseau | Adressage et routage | IP, ICMP |
| 2 — Liaison | Transmission sur le médium | Ethernet, Wi-Fi |
| 1 — Physique | Signaux électriques/optiques | Câbles, fibres, ondes radio |

## Modèle TCP/IP

Le modèle TCP/IP regroupe ces fonctions en quatre couches :

1. **Application** : FTP, FTPS, SFTP, TFTP, SCP, Rsync, HTTP, DNS, etc.
2. **Transport** : TCP, UDP.
3. **Internet** : IP, ICMP.
4. **Accès réseau** : Ethernet, Wi-Fi, ARP.

## Architecture client-serveur

Tous les protocoles présentés reposent sur une architecture client-serveur :

- **Client** : logiciel qui initie la connexion et demande les fichiers.
- **Serveur** : logiciel qui écoute sur un port et fournit les fichiers.
- **Canal de contrôle** : transporte les commandes et réponses.
- **Canal de données** : transporte le contenu des fichiers (FTP/FTPS).
- **Tunnel chiffré** : encapsule contrôle et données (SFTP, SCP, Rsync/SSH).

## Architecture des ports et flux

### FTP classique

```
Client                          Serveur
  | ---- SYN -------------------> |  Port 21 (contrôle)
  | <--- SYN-ACK ----------------- |
  | ---- ACK -------------------> |
  | ---- USER/PASS/CWD/PASV ----> |
  | <--- 227 Entering Passive --- |
  | ---- SYN -------------------> |  Port dynamique (données)
  | <--- SYN-ACK ----------------- |
  | ---- ACK, RETR/PASV data ---> |
```

### SFTP/SCP

```
Client                          Serveur SSH
  | ---- TCP SYN ---------------> |  Port 22
  | <--- SSH banner -------------- |
  | ---- SSH handshake ---------> |
  | <--- Key exchange, auth ----- |
  | ---- SFTP/SCP channel ------> |
  | <--- Données chiffrées ------- |
```

### TFTP

```
Client                          Serveur TFTP
  | ---- RRQ/WRQ (UDP) ---------> |  Port 69
  | <--- DATA/ACK (UDP) ---------- |  Port dynamique
  | ---- ACK/DATA (UDP) --------> |
```

## Architecture de sécurité

- **Confidentialité** : empêcher la lecture par des tiers (chiffrement TLS/SSH).
- **Intégrité** : garantir que le fichier n'est pas altéré (checksums, HMAC).
- **Authentification** : prouver l'identité du client/serveur (mots de passe, clés, certificats).
- **Disponibilité** : assurer le service (redondance, pare-feu, monitoring).
- **Non-répudiation** : traçabilité des transferts (journaux, signatures).

## Architecture moderne : reverse proxy et DMZ

Dans les entreprises, les serveurs de transfert sont souvent placés en DMZ (zone démilitarisée), derrière un reverse proxy ou un bastion SSH :

```
Internet
   |
Pare-feu / WAF
   |
Reverse Proxy (HAProxy, nginx, Traefik)
   |
DMZ — Serveur SFTP/FTPS
   |
Pare-feu interne
   |
Réseau interne / stockage
```

Cette architecture limite l'exposition directe des serveurs et centralise la journalisation.

## Comparaison des architectures

| Protocole | Connexions | Canaux | Transport | Chiffrement | Modèle |
|-----------|------------|--------|-----------|-------------|--------|
| FTP       | 2 (contrôle + données) | Séparés | TCP | Aucun | Client-serveur |
| FTPS      | 2 (contrôle + données) | Séparés, TLS | TCP | TLS/SSL | Client-serveur |
| SFTP      | 1 | Sous-système SSH | TCP sur SSH | SSH | Client-serveur |
| TFTP      | 1 | Simple requête/réponse | UDP | Aucun | Client-serveur |
| SCP       | 1 | Tunnel SSH | TCP sur SSH | SSH | Client-serveur |
| Rsync     | 1 ou 2 | Flux direct ou SSH | TCP | SSH optionnel | Client-serveur / pair-à-pair |


---

# Architecture réseau et modèles de référence

## Modèle OSI

| Couche | Nom | Rôle dans le transfert de fichiers |
|--------|-----|-----------------------------------|
| 7 | Application | FTP, FTPS, SFTP, TFTP, SCP, Rsync, interfaces utilisateur |
| 6 | Présentation | Encodage, compression TLS/SSH, format des données |
| 5 | Session | Gestion des sessions FTP/SFTP, tunnels SSH |
| 4 | Transport | TCP (FTP/FTPS/SFTP/SCP/Rsync), UDP (TFTP) |
| 3 | Réseau | Routage IP, NAT, pare-feu |
| 2 | Liaison | Ethernet, Wi-Fi, switches |
| 1 | Physique | Câbles, fibres, signaux |

## Modèle TCP/IP

| Couche | Protocoles associés |
|--------|---------------------|
| Application | FTP, FTPS, SFTP, TFTP, SCP, Rsync, DNS |
| Transport | TCP, UDP |
| Internet | IP, ICMP, ARP |
| Accès réseau | Ethernet, Wi-Fi, PPP |

## Architecture client-serveur

Tous les protocoles étudiés reposent sur une architecture client-serveur : un hôte central expose un service (serveur), et des hôtes distants (clients) s'y connectent pour envoyer ou récupérer des fichiers. Les échanges peuvent être :

- **Pull** : le client télécharge depuis le serveur.
- **Push** : le client envoie vers le serveur.
- **Peer-to-peer** : possible avec certains outils dérivés (par ex. rsync daemon↔daemon).

## Ports et protocoles transport

| Protocole | Transport | Port(s) par défaut | Remarque |
|-----------|-----------|--------------------|----------|
| FTP       | TCP       | 21 (contrôle), 20 (données actif) | Deux canaux |
| FTPS      | TCP       | 21 (explicite), 990 (implicite) | TLS sur canal contrôle |
| SFTP      | TCP       | 22 | Encapsulé dans SSH |
| TFTP      | UDP       | 69 | Simple, sans session |
| SCP       | TCP       | 22 | Sur tunnel SSH |
| Rsync     | TCP       | 873 (daemon), 22 (sur SSH) | Synchronisation différentielle |

## NAT, pare-feu et modes passif/actif

Le FTP pose des difficultés particulières avec les pare-feu et le NAT à cause de son double canal. Le mode passif (PASV/EPSV) est aujourd'hui le plus répandu car il évite au serveur d'initier une connexion vers le client. FTPS ajoute une complexité supplémentaire car le chiffrement TLS peut masquer les ports dynamiques au pare-feu. SFTP/SCP/Rsync n'utilisent qu'un seul port (22), ce qui simplifie grandement la configuration des pare-feu.

---

# Comparatif rapide des protocoles

| Protocole | Port(s) par défaut | Chiffrement | Authentification | Mode de transport | Cas d'usage typiques |
|-----------|-------------------|-------------|------------------|-------------------|----------------------|
| FTP       | 21 (contrôle), 20 (données actif) | Non         | Login/mot de passe | TCP              | Hébergement web, transferts internes isolés |
| FTPS      | 21 (explicite), 990 (implicite) | TLS/SSL     | Login/mot de passe ou certificat | TCP              | Transferts sécurisés exigeant compatibilité FTP |
| SFTP      | 22                | SSH (chiffrement fort) | Login/mot de passe ou clé SSH | TCP sur SSH      | Administration serveur, transferts sécurisés |
| TFTP      | 69                | Non         | Aucune           | UDP              | Boot réseau, firmware, configuration d'équipements |
| SCP       | 22                | SSH         | Login/mot de passe ou clé SSH | TCP sur SSH      | Copie rapide et simple de fichiers |
| Rsync     | 873 (direct) ou 22 (sur SSH) | SSH si sur SSH | Login/mot de passe ou clé SSH | TCP              | Sauvegardes, synchronisation, miroirs |

Ce tableau permet de choisir rapidement le bon outil selon le contexte : sécurité, simplicité, performance ou compatibilité.

---

# FTP — File Transfer Protocol

## 3.1 Description et historique

FTP (File Transfer Protocol, protocole de transfert de fichiers) est un protocole réseau standardisé utilisé pour transférer des fichiers entre un client et un serveur sur un réseau TCP/IP. Il a été défini pour la première fois en 1971 par Abhay Bhushan sous la forme du RFC 114, puis normalisé dans le RFC 959 en 1985. FTP fonctionne selon un modèle client-serveur et utilise deux canaux de communication distincts : un canal de contrôle pour les commandes et un canal de données pour le transfert effectif des fichiers.

FTP a longtemps été le standard pour le téléchargement et l'envoi de fichiers sur Internet, notamment pour les hébergements web mutualisés, les miroirs de logiciels libres et les échanges inter-organisationnels. Son principal défaut est le manque de chiffrement natif : les identifiants, commandes et données circulent en clair sur le réseau. C'est pourquoi, dans les environnements modernes, FTPS ou SFTP sont généralement préférés pour les transferts sensibles.

## 3.2 Qui a créé FTP ?

- **Abhay Bhushan**, chercheur au MIT, a rédigé le premier RFC (RFC 114) en 1971.
- Le protocole a ensuite été amélioré et standardisé par l'IETF (Internet Engineering Task Force).
- Le RFC 959, publié en octobre 1985, reste la référence fondamentale du protocole FTP.

## 3.3 Où et quand utiliser FTP ?

FTP reste pertinent dans plusieurs situations :

- Hébergement web mutualisé où seul FTP est proposé par l'hébergeur.
- Transfert de fichiers volumineux dans un réseau local isolé ou sécurisé.
- Mise à jour de sites web statiques par des outils comme FileZilla ou Cyberduck.
- Environnements legacy où migrer vers SFTP/FTPS est coûteux.
- Tests et apprentissage réseau en laboratoire contrôlé.

FTP ne doit pas être utilisé sur Internet public sans protection supplémentaire (VPN, FTPS, SFTP) car les identifiants et données sont interceptables.

## 3.4 Architecture et modes de fonctionnement

FTP utilise deux connexions :

1. **Canal de contrôle (control channel)** : port 21, transporte les commandes et réponses.
2. **Canal de données (data channel)** : port 20 en mode actif, ou un port dynamique en mode passif.

### Mode actif (Active Mode)

- Le client se connecte au port 21 du serveur pour le contrôle.
- Le serveur initie la connexion de données vers le client sur le port 20.
- Problème : souvent bloqué par les pare-feu et NAT côté client.

### Mode passif (Passive Mode)

- Le client se connecte au port 21 pour le contrôle.
- Le serveur indique au client un port dynamique à contacter pour les données.
- Le client initie la connexion de données vers ce port.
- Avantage : mieux supporté derrière des pare-feu et NAT.

La commande `PASV` (ou `EPSV` pour IPv6) demande au serveur d'entrer en mode passif.

## 3.5 Ports utilisés

| Fonction        | Port par défaut | Protocole transport |
|-----------------|-----------------|---------------------|
| Contrôle        | 21              | TCP                 |
| Données actif   | 20              | TCP                 |
| Données passif  | plage dynamique | TCP                 |

En mode passif, le serveur FTP ouvre un port éphémère (souvent dans une plage configurée, par exemple 40000-50000) pour le canal de données.

## 3.6 Installation et configuration

### Installation d'un serveur FTP sous Linux (vsftpd)

```bash
# Debian / Ubuntu
sudo apt update
sudo apt install vsftpd -y

# Red Hat / CentOS / Rocky Linux
sudo dnf install vsftpd -y

# Démarrer et activer le service
sudo systemctl start vsftpd
sudo systemctl enable vsftpd
```

### Configuration de base de vsftpd

Le fichier principal de configuration est `/etc/vsftpd.conf`.

```bash
sudo nano /etc/vsftpd.conf
```

Exemple de configuration sécurisée minimale :

```conf
listen=YES
listen_ipv6=NO
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
use_localtime=YES
xferlog_enable=YES
connect_from_port_20=YES
chroot_local_user=YES
allow_writeable_chroot=NO
secure_chroot_dir=/var/run/vsftpd/empty
pam_service_name=vsftpd
rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
ssl_enable=NO
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=50000
pasv_address=192.168.1.10
```

Redémarrer le service après modification :

```bash
sudo systemctl restart vsftpd
```

### Installation d'un client FTP sous Windows

- FileZilla Client : `https://filezilla-project.org/`
- WinSCP : `https://winscp.net/`
- Commande intégrée `ftp` dans l'invite de commandes Windows (optionnal features).

Pour activer le client FTP natif Windows via PowerShell :

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName TFTP,TelnetClient -All
```

## 3.7 Commandes FTP essentielles

| Commande | Description |
|----------|-------------|
| `open hote`     | Ouvrir une connexion vers un hôte |
| `user nom`      | Saisir l'utilisateur |
| `pass mot`      | Saisir le mot de passe |
| `ls`            | Lister les fichiers |
| `dir`           | Lister avec détails |
| `cd repertoire` | Changer de répertoire distant |
| `lcd repertoire`| Changer de répertoire local |
| `get fichier`   | Télécharger un fichier |
| `mget fichiers` | Télécharger plusieurs fichiers |
| `put fichier`   | Envoyer un fichier |
| `mput fichiers` | Envoyer plusieurs fichiers |
| `binary`        | Mode binaire |
| `ascii`         | Mode ASCII |
| `passive`       | Activer le mode passif |
| `active`        | Activer le mode actif |
| `mkdir rep`     | Créer un répertoire |
| `rmdir rep`     | Supprimer un répertoire |
| `delete fichier`| Supprimer un fichier |
| `bye` / `quit`  | Quitter la session |

## 3.8 Vérification / dépannage sous Linux et Windows

### Vérifier que le service FTP écoute

Sous Linux :

```bash
sudo ss -tlnp | grep -E '21|vsftpd'
sudo systemctl status vsftpd
```

Sous Windows (si un serveur FTP est installé) :

```powershell
Get-NetTCPConnection -LocalPort 21 | Select-Object LocalAddress, LocalPort, State, OwningProcess
Get-Process -Id (Get-NetTCPConnection -LocalPort 21).OwningProcess
```

### Tester la connectivité réseau

Sous Linux :

```bash
telnet serveur-ftp.example.com 21
nc -vz serveur-ftp.example.com 21
```

Sous Windows :

```powershell
Test-NetConnection -ComputerName serveur-ftp.example.com -Port 21
```

### Analyser le trafic FTP

Sous Linux avec tcpdump :

```bash
sudo tcpdump -i any port 21 or port 20 -nn -A
```

Avec Wireshark, filtrer sur `ftp` ou `ftp-data`.

### Journaux FTP

Sous Linux (vsftpd) :

```bash
sudo tail -f /var/log/vsftpd.log
sudo tail -f /var/log/syslog | grep vsftpd
```

Sous Windows (IIS FTP) :

```powershell
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-IIS-Logging/FTPSVC'; ID=1} | Select-Object -First 20
```

## 3.9 Envoi et réception de fichiers

### Transfert interactif en ligne de commande

Sous Linux :

```bash
ftp serveur-ftp.example.com
# puis saisir login et mot de passe
ftp> passive
ftp> binary
ftp> get fichier distant local
ftp> put local distant
ftp> mget fichiers
ftp> mput fichiers
ftp> bye
```

Sous Windows (cmd) :

```cmd
ftp serveur-ftp.example.com
ftp> user monlogin
ftp> pass monmotdepasse
ftp> passive
ftp> get fichier.txt
ftp> put fichier.txt
ftp> quit
```

### Transfert automatisé par script

Créer un fichier `ftp_commands.txt` :

```txt
open serveur-ftp.example.com
user monlogin monmotdepasse
binary
passive
lcd /chemin/local
cd /chemin/distant
get fichier.zip
put rapport.pdf
bye
```

Exécuter :

```bash
ftp -n < ftp_commands.txt
```

Sous Windows :

```cmd
ftp -s:ftp_commands.txt
```

### Envoi récursif avec un client graphique

FileZilla permet de glisser-déposer des arborescences complètes, de gérer les files d'attente, les reprises de transfert et les permissions.

## 3.10 Sécurité et bonnes pratiques

- Désactiver l'accès anonyme (`anonymous_enable=NO`).
- Restreindre les utilisateurs à leur répertoire personnel (`chroot_local_user=YES`).
- Utiliser FTPS ou SFTP plutôt que FTP sur Internet.
- Limiter la plage de ports passifs et ouvrir ces ports au pare-feu.
- Surveiller les journaux et mettre à jour le serveur.
- Bannir les adresses IP après trop de tentatives échouées (fail2ban).
- Ne jamais stocker les mots de passe en clair dans des scripts automatisés.

Exemple fail2ban pour vsftpd :

```bash
sudo apt install fail2ban
sudo nano /etc/fail2ban/jail.local
```

```ini
[vsftpd]
enabled  = true
port     = ftp,ftp-data,ftps,ftps-data
filter   = vsftpd
logpath  = /var/log/vsftpd.log
maxretry = 5
bantime  = 3600
```

## 3.11 Outils clients FTP

- **FileZilla** : multiplateforme, gratuit, très répandu.
- **Cyberduck** : macOS / Windows, interface soignée.
- **WinSCP** : Windows, support FTP/FTPS/SFTP/SCP.
- **lftp** : client en ligne de commande puissant sous Linux.
- **ncftp** : client FTP avancé en ligne de commande.
- **gFTP** : client graphique Linux.

Exemple lftp :

```bash
lftp -u login,password ftp://serveur.example.com
lftp> mirror /distant /local
```

## 3.12 Ressources et livres

- RFC 959 — File Transfer Protocol (FTP) : `https://tools.ietf.org/html/rfc959`
- RFC 3659 — Extensions to FTP : `https://tools.ietf.org/html/rfc3659`
- Site officiel vsftpd : `https://security.appspot.com/vsftpd.html`
- Livre : *The TCP/IP Guide* par Charles M. Kozierok
- Livre : *Networking All-in-One For Dummies* par Doug Lowe

---

# FTPS — FTP over SSL/TLS

## 4.1 Description et historique

FTPS (FTP over SSL/TLS) désigne l'encapsulation du protocole FTP dans une couche TLS/SSL afin de chiffrer les canaux de contrôle et/ou de données. Il ne doit pas être confondu avec SFTP, qui repose sur SSH. FTPS a été standardisé principalement par le RFC 2228 (commandes de sécurité FTP) et le RFC 4217 (FTP over TLS), publié en 2005.

FTPS permet de sécuriser les transferts tout en conservant la logique et la compatibilité de FTP. C'est une solution courante dans les échanges B2B (business-to-business), les systèmes bancaires et les hébergeurs proposant du FTP sécurisé.

## 4.2 Différences FTP vs FTPS

| Aspect | FTP | FTPS |
|--------|-----|------|
| Chiffrement | Aucun | TLS/SSL |
| Ports | 21 / 20 | 21 (explicite), 990 (implicite) |
| Sécurité identifiants | Faible | Forte |
| Certificats | Non | Oui |
| Compatibilité | Universelle | Nécessite client/serveur compatibles |

## 4.3 Modes FTPS : explicite et implicite

### FTPS explicite (FTPES)

- La connexion commence en clair sur le port 21.
- Le client envoie la commande `AUTH TLS` pour négocier TLS.
- Ensuite, les commandes et données sont chiffrées.
- Plus flexible et compatible avec les pare-feu.

### FTPS implicite

- La négociation TLS est implicite dès la connexion.
- Le port par défaut est 990 pour le contrôle et 989 pour les données.
- Moins utilisé aujourd'hui, considéré comme déprécié par certains.

## 4.4 Configuration serveur FTPS

Avec vsftpd, activer SSL/TLS :

```conf
ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
require_ssl_reuse=NO
ssl_ciphers=HIGH
rsa_cert_file=/etc/ssl/certs/vsftpd.pem
rsa_private_key_file=/etc/ssl/private/vsftpd.key
implicit_ssl=NO
listen_port=21
```

Générer un certificat auto-signé (test uniquement) :

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/vsftpd.key \
  -out /etc/ssl/certs/vsftpd.pem \
  -subj "/C=TD/ST=Chad/L=N'Djamena/O=hackers_tchad/CN=ftp.example.com"
```

Redémarrer vsftpd :

```bash
sudo systemctl restart vsftpd
```

## 4.5 Vérification sous Linux et Windows

### Vérifier le port FTPS

Sous Linux :

```bash
sudo ss -tlnp | grep -E '21|990|vsftpd'
openssl s_client -connect serveur.example.com:21 -starttls ftp
openssl s_client -connect serveur.example.com:990
```

Sous Windows avec PowerShell :

```powershell
Test-NetConnection -ComputerName serveur.example.com -Port 21
Test-NetConnection -ComputerName serveur.example.com -Port 990
```

### Vérifier le certificat

```bash
echo | openssl s_client -connect serveur.example.com:21 -starttls ftp 2>/dev/null | openssl x509 -noout -dates -subject -issuer
```

## 4.6 Commandes et opérations

Avec lftp en FTPS explicite :

```bash
lftp -u login,password -e "set ftp:ssl-force true; set ssl:verify-certificate no; ls; bye" ftps://serveur.example.com
```

Avec curl :

```bash
curl -v --ftp-ssl --user login:password ftp://serveur.example.com/fichier.txt -o fichier.txt
```

Avec FileZilla : choisir le protocole `FTP - File Transfer Protocol`, puis `Exiger FTP explicite sur TLS`.

## 4.7 Bonnes pratiques

- Toujours utiliser FTPS explicite plutôt qu'implicite.
- Utiliser des certificats signés par une autorité de confiance (Let's Encrypt, etc.).
- Vérifier la chaîne de certificats côté client.
- Désactiver les anciennes versions TLS (TLS 1.0, 1.1) si possible.
- Limiter la plage de ports passifs et configurer le pare-feu en conséquence.

---

# SFTP — SSH File Transfer Protocol

## 5.1 Description et historique

SFTP (SSH File Transfer Protocol), parfois appelé Secure FTP, est un protocole de transfert de fichiers qui s'appuie sur SSH (Secure Shell). Contrairement à FTPS, qui sécurise FTP avec TLS, SFTP est un sous-système intégré à SSH et utilise un seul canal chiffré sur le port 22.

SFTP a été conçu par Tatu Ylönen et son équipe au milieu des années 1990, en même temps que SSH. Il offre une gestion complète des fichiers distants : lister, télécharger, envoyer, renommer, supprimer, gérer les permissions, les liens symboliques, etc.

## 5.2 Différences SFTP vs FTPS vs SCP

| Aspect | SFTP | FTPS | SCP |
|--------|------|------|-----|
| Protocole sous-jacent | SSH | FTP + TLS | SSH |
| Port | 22 | 21/990 | 22 |
| Commandes riches | Oui | Oui | Non (copie seule) |
| Reprise d'upload | Oui | Dépend du client | Non |
| Passe facilement les pare-feu | Oui | Parfois difficile (ports passifs) | Oui |
| Authentification par clé | Oui | Certificats client possibles | Oui |

## 5.3 Installation et configuration

### Serveur SFTP (OpenSSH)

Sous Linux, OpenSSH est généralement préinstallé. Pour l'installer :

```bash
# Debian / Ubuntu
sudo apt install openssh-server -y

# Red Hat / CentOS / Rocky
sudo dnf install openssh-server -y

sudo systemctl start sshd
sudo systemctl enable sshd
```

Le fichier de configuration est `/etc/ssh/sshd_config`.

### Configurer un chroot SFTP

Pour restreindre un utilisateur à un répertoire :

```bash
sudo nano /etc/ssh/sshd_config
```

Ajouter :

```conf
Match Group sftpusers
    ChrootDirectory /srv/sftp/%u
    ForceCommand internal-sftp
    AllowTcpForwarding no
    X11Forwarding no
    PasswordAuthentication yes
```

Créer l'utilisateur et le répertoire :

```bash
sudo groupadd sftpusers
sudo useradd -m -G sftpusers -s /sbin/nologin sftpuser
sudo mkdir -p /srv/sftp/sftpuser/upload
sudo chown root:root /srv/sftp/sftpuser
sudo chmod 755 /srv/sftp/sftpuser
sudo chown sftpuser:sftpusers /srv/sftp/sftpuser/upload
sudo systemctl restart sshd
```

## 5.4 Authentification par clé SSH

Générer une paire de clés :

```bash
ssh-keygen -t ed25519 -C "hackers_tchad@sftp.example.com"
```

Copier la clé publique sur le serveur :

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub sftpuser@serveur.example.com
```

Se connecter sans mot de passe :

```bash
sftp -i ~/.ssh/id_ed25519 sftpuser@serveur.example.com
```

## 5.5 Commandes SFTP

| Commande | Description |
|----------|-------------|
| `sftp user@hote` | Connexion interactive |
| `ls`             | Lister les fichiers distants |
| `lls`            | Lister les fichiers locaux |
| `cd rep`         | Changer de répertoire distant |
| `lcd rep`        | Changer de répertoire local |
| `get fichier`    | Télécharger |
| `get -r rep`     | Télécharger un répertoire |
| `put fichier`    | Envoyer |
| `put -r rep`     | Envoyer un répertoire |
| `mkdir rep`      | Créer un répertoire distant |
| `rm fichier`     | Supprimer un fichier distant |
| `rmdir rep`      | Supprimer un répertoire distant |
| `chmod mode fichier` | Changer les permissions |
| `bye` / `quit`   | Quitter |

## 5.6 Vérification sous Linux et Windows

### Vérifier le service SSH/SFTP

Sous Linux :

```bash
sudo systemctl status sshd
sudo ss -tlnp | grep :22
```

Sous Windows (si OpenSSH Serveur est installé) :

```powershell
Get-Service sshd
Test-NetConnection -ComputerName serveur.example.com -Port 22
```

### Tester SFTP en ligne de commande

Sous Linux et Windows (avec client SFTP installé) :

```bash
sftp sftpuser@serveur.example.com
sftp -i ~/.ssh/id_ed25519 sftpuser@serveur.example.com
```

### Vérifier la version du serveur SSH

```bash
ssh -V
```

### Auditer la configuration SSH

```bash
sudo sshd -t
```

## 5.7 Scripts et automatisation

### Script SFTP batch

Fichier `sftp_batch.txt` :

```txt
cd /distant
lcd /local
put fichier.txt
get rapport.csv
bye
```

Exécution :

```bash
sftp -b sftp_batch.txt -i ~/.ssh/id_ed25519 sftpuser@serveur.example.com
```

### Automatisation Python avec Paramiko

```python
import paramiko

transport = paramiko.Transport(("serveur.example.com", 22))
transport.connect(username="sftpuser", password="secret")
sftp = paramiko.SFTPClient.from_transport(transport)
sftp.put("local.txt", "/remote/local.txt")
sftp.get("/remote/remote.txt", "local_remote.txt")
sftp.close()
transport.close()
```

## 5.8 Sécurité

- Désactiver l'authentification par mot de passe si possible (`PasswordAuthentication no`).
- Utiliser des clés Ed25519 ou RSA 4096 bits.
- Restreindre les utilisateurs avec un chroot.
- Mettre à jour régulièrement OpenSSH.
- Surveiller les journaux `/var/log/auth.log` ou `/var/log/secure`.
- Utiliser fail2ban pour bloquer les attaques par force brute.

---

# TFTP — Trivial File Transfer Protocol

## 6.1 Description et historique

TFTP (Trivial File Transfer Protocol) est un protocole de transfert de fichiers très simple, défini dans le RFC 1350 en 1992 (remplaçant le RFC 783 de 1981). Il repose sur UDP (port 69) et n'offre ni authentification, ni chiffrement, ni liste de répertoires. TFTP est principalement utilisé pour le démarrage réseau (PXE), le chargement de firmware sur des équipements réseau, et la sauvegarde/restauration de configurations de routeurs/switchs.

## 6.2 Cas d'usage

- Boot PXE de postes de travail sans disque.
- Mise à jour de firmware de routeurs, switchs, téléphones IP.
- Sauvegarde de configurations Cisco, Juniper, etc.
- Déploiement d'images système en environnement contrôlé.

## 6.3 Installation et configuration

### Serveur TFTP sous Linux (tftpd-hpa)

```bash
sudo apt install tftpd-hpa -y
sudo systemctl start tftpd-hpa
sudo systemctl enable tftpd-hpa
```

Configuration dans `/etc/default/tftpd-hpa` :

```conf
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/srv/tftp"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="--secure --create"
```

Créer le répertoire :

```bash
sudo mkdir -p /srv/tftp
sudo chown tftp:tftp /srv/tftp
sudo chmod 777 /srv/tftp
sudo systemctl restart tftpd-hpa
```

### Client TFTP sous Windows

Activer le client TFTP optionnel :

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName TFTP -All
```

## 6.4 Commandes TFTP

Sous Linux :

```bash
tftp serveur.example.com
 tftp> get fichier.txt
 tftp> put fichier.txt
 tftp> quit
```

Sous Windows :

```cmd
tftp -i serveur.example.com GET fichier.txt C:\local\fichier.txt
tftp -i serveur.example.com PUT C:\local\fichier.txt fichier.txt
```

## 6.5 Vérification sous Linux et Windows

Sous Linux :

```bash
sudo ss -ulnp | grep :69
sudo systemctl status tftpd-hpa
sudo ls -la /srv/tftp
```

Sous Windows :

```powershell
Get-NetUDPEndpoint -LocalPort 69 | Select-Object LocalAddress, LocalPort, OwningProcess
```

## 6.6 Limites et sécurité

- Aucune authentification : ne pas exposer sur Internet.
- Aucun chiffrement : utiliser uniquement sur réseaux isolés et de confiance.
- Limité à des transferts de fichiers simples.
- Configurer les permissions en lecture seule si possible.
- Limiter l'accès par pare-feu à quelques IP autorisées.

---

# SCP — Secure Copy Protocol

## 7.1 Description et historique

SCP (Secure Copy Protocol) est un protocole de copie de fichiers sécurisé basé sur SSH. Il permet de transférer des fichiers entre deux machines de manière simple et chiffrée. Historiquement, SCP a été développé en tant qu'outil de la suite OpenSSH. Il repose sur le protocole RCP (Remote Copy Protocol) mais avec un tunnel SSH.

## 7.2 Syntaxe SCP

```bash
# Envoyer un fichier local vers un serveur distant
scp fichier.txt user@serveur.example.com:/chemin/distant/

# Récupérer un fichier distant
scp user@serveur.example.com:/chemin/distant/fichier.txt ./local/

# Transfert récursif de répertoire
scp -r repertoire/ user@serveur.example.com:/chemin/distant/

# Spécifier un port SSH non standard
scp -P 2222 fichier.txt user@serveur.example.com:/chemin/

# Préserver les permissions et horodatages
scp -p fichier.txt user@serveur.example.com:/chemin/

# Transfert entre deux hôtes distants
scp user1@hote1:/fichier user2@hote2:/dest/
```

## 7.3 Exemples avancés

### Compression pendant le transfert

```bash
scp -C gros_fichier.tar.gz user@serveur.example.com:/backup/
```

### Transfert avec clé SSH spécifique

```bash
scp -i ~/.ssh/id_ed25519 fichier.txt user@serveur.example.com:/home/user/
```

### Transfert limité en bande passante

```bash
scp -l 1000 gros_fichier.iso user@serveur.example.com:/data/
```

## 7.4 Vérification sous Linux et Windows

Sous Linux :

```bash
which scp
scp -V
ssh -V
```

Sous Windows (PowerShell avec OpenSSH installé) :

```powershell
Get-Command scp
scp -V
```

## 7.5 Remplacement par SFTP / Rsync

SCP est simple mais peu flexible (pas de reprise sur interruption, pas de synchronisation différentielle). OpenSSH recommande progressivement d'utiliser SFTP ou Rsync pour les transferts complexes.

---

# Rsync

## 8.1 Description et historique

Rsync est un outil de synchronisation de fichiers développé par Andrew Tridgell et Paul Mackerras en 1996. Il est particulièrement efficace car il ne transfère que les différences entre la source et la destination (algorithme de rolling checksum), ce qui réduit considérablement le temps et la bande passante nécessaires.

## 8.2 Algorithmes et optimisation

Rsync utilise une combinaison de :

- **Checksums** (MD5/SHA1 selon version) pour détecter les blocs modifiés.
- **Rolling checksum** pour identifier les blocs inchangés.
- **Compression optionnelle** avec `-z`.
- **Transfert incrémental** pour les gros fichiers.

## 8.3 Installation

Sous Linux :

```bash
sudo apt install rsync -y
sudo dnf install rsync -y
```

Sous Windows :

- Installer via Cygwin, MSYS2, ou WSL (Windows Subsystem for Linux).
- Utilitaires graphiques : DeltaCopy, Grsync.

## 8.4 Syntaxe et options

```bash
# Synchronisation locale
rsync -av /source/ /destination/

# Synchronisation distante via SSH
rsync -avz -e ssh /source/ user@serveur.example.com:/destination/

# Synchronisation avec suppression des fichiers absents de la source
rsync -av --delete /source/ /destination/

# Afficher la progression
rsync -av --progress /source/ user@serveur.example.com:/destination/

# Exclure des fichiers
rsync -av --exclude='*.tmp' /source/ /destination/

# Simulation (dry-run)
rsync -avn /source/ /destination/
```

Options courantes :

- `-a` : archive (préserve permissions, liens, horodatages, etc.)
- `-v` : verbeux
- `-z` : compression
- `-r` : récursif
- `-P` : affiche la progression et permet la reprise
- `--delete` : supprime les fichiers absents de la source
- `-e ssh` : utilise SSH comme transport

## 8.5 Synchronisation locale et distante

### Sauvegarde locale incrémentale

```bash
rsync -av --delete /home/user/documents/ /backup/documents/
```

### Sauvegarde distante

```bash
rsync -avz --progress -e "ssh -i ~/.ssh/id_ed25519" \
  /home/user/important/ user@serveur.example.com:/backups/important/
```

## 8.6 Intégration SSH

Rsync peut utiliser SSH comme transport sous-jacent, ce qui sécurise la synchronisation.

```bash
rsync -avz -e "ssh -p 2222 -i ~/.ssh/id_rsa" /source/ user@hote:/dest/
```

## 8.7 Vérification sous Linux et Windows

Sous Linux :

```bash
rsync --version
which rsync
```

Sous Windows (via WSL) :

```bash
wsl rsync --version
```

## 8.8 Sauvegardes et rotations

Script de sauvegarde quotidienne avec rotation :

```bash
#!/bin/bash
SOURCE="/home/user/data"
DEST="/backup/data"
DATE=$(date +%Y%m%d)
rsync -av --delete "$SOURCE/" "$DEST/current/"
cp -al "$DEST/current" "$DEST/$DATE"
find "$DEST" -maxdepth 1 -type d -mtime +7 -exec rm -rf {} \;
```

---

# Synthèse des commandes de vérification

## Linux

```bash
# FTP
sudo ss -tlnp | grep -E '21|20'
sudo systemctl status vsftpd

# FTPS
openssl s_client -connect hote:21 -starttls ftp
sudo ss -tlnp | grep -E '21|990'

# SFTP
sudo systemctl status sshd
sudo ss -tlnp | grep :22
ssh -V

# TFTP
sudo ss -ulnp | grep :69
sudo systemctl status tftpd-hpa

# SCP
which scp
scp -V

# Rsync
rsync --version
```

## Windows

```powershell
# FTP / FTPS
Test-NetConnection -ComputerName hote -Port 21
Test-NetConnection -ComputerName hote -Port 990

# SFTP / SCP / Rsync sur SSH
Test-NetConnection -ComputerName hote -Port 22

# TFTP
Get-NetUDPEndpoint -LocalPort 69 | Select-Object LocalAddress, LocalPort, OwningProcess

# Services OpenSSH / IIS FTP
Get-Service sshd
Get-WindowsFeature Web-FTP-Server
```

---

# Applications et outils graphiques

| Outil        | Plateforme       | Protocoles supportés | Particularités |
|--------------|------------------|----------------------|----------------|
| FileZilla    | Windows, Linux, macOS | FTP, FTPS, SFTP | Gratuit, très populaire |
| WinSCP       | Windows          | FTP, FTPS, SFTP, SCP, WebDAV | Puissant, scripting |
| Cyberduck    | Windows, macOS   | FTP, FTPS, SFTP, WebDAV, S3 | Interface épurée |
| ForkLift     | macOS            | FTP, FTPS, SFTP, S3, AFP | Gestionnaire de fichiers dual pane |
| Transmit     | macOS            | FTP, FTPS, SFTP, S3 | Professionnel, rapide |
| lftp         | Linux            | FTP, FTPS, SFTP, HTTP, BitTorrent | Ligne de commande avancée |
| rsync / grsync | Linux, Windows | rsync, SSH | Synchronisation efficace |
| SolarWinds TFTP Server | Windows | TFTP | Serveur TFTP gratuit |
| Tftpd64      | Windows          | TFTP, DHCP, SNTP, Syslog | Outil réseau complet |

---

# Livres, ressources et liens

## Livres recommandés

- *The TCP/IP Guide* — Charles M. Kozierok
- *SSH, The Secure Shell: The Definitive Guide* — Daniel J. Barrett, Richard E. Silverman, Robert G. Byrnes
- *UNIX and Linux System Administration Handbook* — Evi Nemeth et al.
- *Linux Command Line and Shell Scripting Bible* — Richard Blum, Christine Bresnahan
- *Networking All-in-One For Dummies* — Doug Lowe
- *Practical Packet Analysis* — Chris Sanders

## RFC et standards

- RFC 959 — FTP : `https://tools.ietf.org/html/rfc959`
- RFC 2228 — FTP Security Extensions : `https://tools.ietf.org/html/rfc2228`
- RFC 4217 — FTP over TLS : `https://tools.ietf.org/html/rfc4217`
- RFC 4251 — SSH Protocol Architecture : `https://tools.ietf.org/html/rfc4251`
- RFC 4253 — SSH Transport Layer Protocol : `https://tools.ietf.org/html/rfc4253`
- RFC 1350 — TFTP : `https://tools.ietf.org/html/rfc1350`
- Rsync algorithm : `https://rsync.samba.org/tech_report/`

## Liens utiles

- OpenSSH : `https://www.openssh.com/`
- FileZilla : `https://filezilla-project.org/`
- WinSCP : `https://winscp.net/`
- Cyberduck : `https://cyberduck.io/`
- Rsync : `https://rsync.samba.org/`
- vsftpd : `https://security.appspot.com/vsftpd.html`
- Proftpd : `http://www.proftpd.org/`
- Tftpd-hpa : `https://git.kernel.org/pub/scm/network/tftp/tftp-hpa.git/`

---

# Glossaire

- **B2B** : Business-to-Business, échange entre entreprises.
- **Chroot** : Changement de racine, isolation d'un utilisateur dans un répertoire.
- **CLI** : Command Line Interface, interface en ligne de commande.
- **FTP** : File Transfer Protocol.
- **FTPS** : FTP over SSL/TLS.
- **NAT** : Network Address Translation.
- **PXE** : Preboot eXecution Environment, démarrage réseau.
- **SCP** : Secure Copy Protocol.
- **SFTP** : SSH File Transfer Protocol.
- **SSH** : Secure Shell.
- **SSL/TLS** : Secure Sockets Layer / Transport Layer Security.
- **TFTP** : Trivial File Transfer Protocol.
- **UDP** : User Datagram Protocol.

---

# Conclusion

Ce guide couvre les principaux protocoles de transfert de fichiers utilisés en 2026 : FTP, FTPS, SFTP, TFTP, SCP et Rsync. Chaque protocole a ses forces et ses faiblesses. Le choix dépend du contexte : sécurité, simplicité, performance, compatibilité ou environnement legacy.

**Recommandations générales :**

- Pour les transferts sécurisés modernes : privilégier **SFTP** ou **Rsync sur SSH**.
- Pour les échanges nécessitant la compatibilité FTP : utiliser **FTPS explicite**.
- Pour le démarrage réseau ou les équipements embarqués : utiliser **TFTP** sur un réseau isolé.
- Pour les copies simples et rapides : utiliser **SCP**.
- Éviter FTP en clair sur Internet ou sur des réseaux non sécurisés.

**Auteur :** `hackers_tchad`  
**Date :** 2026  
**Version :** 1.0  
**Objectif :** Référence éducative et opérationnelle pour administrateurs, développeurs et passionnés de cybersécurité.

---

*Fin du document.*
