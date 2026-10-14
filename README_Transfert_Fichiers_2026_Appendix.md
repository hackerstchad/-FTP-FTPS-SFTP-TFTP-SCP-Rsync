
---

# Annexes avancées

## A. Benchmarks et performances

### Comparaison théorique des vitesses

| Protocole | Overhead | Reprise | Compression | Latence | Usage CPU |
|-----------|----------|---------|-------------|---------|-----------|
| FTP       | Faible   | Non     | Non         | Moyenne | Faible    |
| FTPS      | Moyen    | Non     | Non         | Moyenne | Moyen     |
| SFTP      | Moyen    | Oui     | Non         | Faible  | Moyen     |
| TFTP      | Très faible | Non  | Non         | Faible  | Très faible |
| SCP       | Faible   | Non     | Oui (-C)    | Faible  | Faible    |
| Rsync     | Variable | Oui     | Oui (-z)    | Faible  | Moyen     |

### Tests de débit avec iperf3 / rsync

```bash
# Générer un fichier de test
dd if=/dev/urandom of=/tmp/test.bin bs=1M count=1024

# Mesurer le temps de transfert SCP
time scp /tmp/test.bin user@serveur:/tmp/

# Mesurer rsync avec compression
time rsync -avz /tmp/test.bin user@serveur:/tmp/

# Mesurer SFTP batch
echo "put /tmp/test.bin /tmp/test.bin" | time sftp user@serveur
```

## B. Sécurité avancée

### Menaces courantes

- **Sniffing** : interception des identifiants en clair (FTP, TFTP).
- **Man-in-the-middle** : usurpation du serveur (FTP, FTPS mal configuré).
- **Force brute** : attaques sur mots de passe SSH/SFTP.
- **Exfiltration** : transferts non autorisés via comptes compromis.
- **Élévation de privilèges** : mauvaise configuration du chroot SFTP.

### Bonnes pratiques 2026

1. Toujours privilégier SFTP ou Rsync sur SSH.
2. Désactiver l'authentification par mot de passe SSH.
3. Utiliser des clés Ed25519 ou RSA ≥ 4096 bits.
4. Restreindre les comptes de service avec chroot et commandes forcées.
5. Activer l'authentification multifacteur (MFA) sur SSH.
6. Surveiller les journaux avec SIEM / fail2ban.
7. Segmenter le réseau (VLAN, ACL) pour les services TFTP/FTP legacy.
8. Maintenir à jour OpenSSH, OpenSSL, vsftpd, ProFTPD, Rsync.
9. Auditer régulièrement les certificats TLS et leur expiration.
10. Chiffrer les données au repos en plus du transport si nécessaire.

### Configuration fail2ban pour SSH/SFTP

```ini
# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
```

## C. Scripts d'automatisation

### Sauvegarde complète avec rsync + SFTP

```bash
#!/bin/bash
# backup.sh
SOURCE=/home/user/data
DEST=user@backup.example.com:/backups/$(hostname)
KEY=~/.ssh/backup_ed25519
LOG=/var/log/backup.log

rsync -avz --delete -e "ssh -i $KEY" "$SOURCE/" "$DEST/current/" >> "$LOG" 2>&1
if [ $? -eq 0 ]; then
  echo "[OK] Sauvegarde terminée $(date)" >> "$LOG"
else
  echo "[ERREUR] Échec de la sauvegarde $(date)" >> "$LOG"
fi
```

### Scan de ports FTP/SFTP/FTPS en Python

```python
import socket

def test_port(host, port, proto=socket.SOCK_STREAM):
    try:
        s = socket.socket(socket.AF_INET, proto)
        s.settimeout(2)
        s.connect((host, port))
        s.close()
        return True
    except Exception:
        return False

services = {'FTP':21,'FTPS-implicite':990,'SFTP/SSH':22,'Rsync':873}
for name, port in services.items():
    status = 'OUVERT' if test_port('example.com', port) else 'FERMÉ'
    print(f"{name} (port {port}) : {status}")
```

## D. Intégration CI/CD et DevOps

### Déploiement par SFTP dans GitHub Actions

```yaml
name: Deploy via SFTP
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: |
          echo "put -r ./public /var/www/site" | \
          sftp -i <(echo "$SSH_KEY") -o StrictHostKeyChecking=no user@serveur
```

### Transfert avec Rsync dans GitLab CI

```yaml
deploy:
  stage: deploy
  image: alpine/rsync
  script:
    - rsync -avz --delete -e "ssh -i $SSH_KEY -o StrictHostKeyChecking=no" ./dist/ user@serveur:/var/www/
```

## E. Cloud et stockage objet

Les protocoles traditionnels coexistent avec les API cloud :

- **Amazon S3** : API HTTP/HTTPS, outils comme `aws s3`, `s3cmd`, `rclone`.
- **Azure Blob Storage** : API REST, `azcopy`, `rclone`.
- **Google Cloud Storage** : `gsutil`, `rclone`.
- **Rclone** : outil universel supportant SFTP, FTPS, S3, Azure, GCS, Dropbox, etc.

### Exemple rclone

```bash
rclone config create mysftp sftp host=serveur.example.com user=hackers_tchad
rclone sync /local/path mysftp:/remote/path -P
```

## F. Virtualisation et conteneurs

### Serveur SFTP dans Docker

```dockerfile
FROM atmoz/sftp:latest
COPY users.conf /etc/sftp/users.conf
EXPOSE 22
```

```bash
docker run -p 22:22 -v /host/share:/home/hackers_tchad/share atmoz/sftp hackers_tchad:pass:1001
```

### Serveur FTP avec vsftpd sous Docker

```bash
docker run -d -v /host/ftp:/home/vsftpd -p 21:21 -p 40000-40010:40000-40010 fauria/vsftpd
```

## G. Monitoring et supervision

### Métriques clés

- Disponibilité du service (uptime).
- Nombre de connexions actives.
- Volume de données transférées.
- Temps de transfert moyen.
- Échecs d'authentification.
- Certificats TLS proches de l'expiration.

### Outils de monitoring

- **Prometheus + Grafana** : métriques temps réel.
- **Nagios / Zabbix** : alertes de disponibilité.
- **ELK Stack** : centralisation des logs.
- **Nethogs / iftop** : analyse du trafic par processus.

## H. Conformité et normes

- **RGPD** : chiffrement des transferts de données personnelles.
- **ISO 27001** : contrôles d'accès et traçabilité.
- **HIPAA** : sécurisation des échanges de santé (USA).
- **PCI-DSS** : transferts de données de paiement chiffrés.
- **NIST** : recommandations sur SSH/TLS.

## I. Histoire et chronologie

| Année | Événement |
|-------|-----------|
| 1971 | RFC 114, premier FTP |
| 1985 | RFC 959, FTP standardisé |
| 1991 | PGP, premières réflexions sur la sécurité des transferts |
| 1992 | RFC 1350, TFTP |
| 1996 | Création de rsync |
| 1999 | SSL 3.0 / TLS 1.0 |
| 2001 | RFC 4217, FTPS |
| 2006 | RFC 4344, SSH File Transfer Protocol v6 |
| 2018 | TLS 1.3 publié (RFC 8446) |
| 2026 | Ce guide, avec l'état de l'art actuel |

## J. Ressources complémentaires

### Cours en ligne

- Coursera : Computer Networking (Stanford).
- Udemy : Linux Administration, SSH Mastery.
- TryHackMe / Hack The Box : labs réseau et protocoles.

### Chaînes et communautés

- r/sysadmin, r/linuxadmin, r/cybersecurity.
- NetworkChuck, David Bombal (YouTube).
- hackers_tchad — veille cybersécurité.

### Livres additionnels

- *TCP/IP Illustrated, Vol. 1* — W. Richard Stevens.
- *Applied Cryptography* — Bruce Schneier.
- *Network Security Assessment* — Chris McNab.
- *Linux Firewalls* — Steve Suehring.
- *Automating Linux and Unix System Administration* — Kirk Bauer.

### Liens officiels supplémentaires

- TLS 1.3 RFC 8446 : https://tools.ietf.org/html/rfc8446
- OpenSSL : https://www.openssl.org/
- Proftpd : http://www.proftpd.org/
- Samba / Rsync : https://rsync.samba.org/
- Tftpd-hpa : https://git.kernel.org/pub/scm/network/tftp/tftp-hpa.git/
- FileZilla : https://filezilla-project.org/
- WinSCP : https://winscp.net/
- Cyberduck : https://cyberduck.io/
- lftp : https://lftp.yar.ru/
- rclone : https://rclone.org/

## K. Glossaire complémentaire

- **ACL** : Access Control List.
- **AES** : Advanced Encryption Standard.
- **CA** : Certificate Authority.
- **DLP** : Data Loss Prevention.
- **DMZ** : Demilitarized Zone.
- **EDR** : Endpoint Detection and Response.
- **HMAC** : Hash-based Message Authentication Code.
- **IDS/IPS** : Intrusion Detection/Prevention System.
- **MFA** : Multi-Factor Authentication.
- **PKI** : Public Key Infrastructure.
- **SIEM** : Security Information and Event Management.
- **VLAN** : Virtual LAN.
- **VPN** : Virtual Private Network.
- **WAF** : Web Application Firewall.

## L. Conclusion avancée

La maîtrise des protocoles de transfert de fichiers est indispensable pour tout administrateur système, ingénieur réseau ou professionnel de la cybersécurité. En 2026, l'écosystème est dominé par SSH/SFTP et Rsync pour leur sécurité et leur flexibilité, tandis que FTP et TFTP survivent dans des niches legacy ou embarquées. FTPS reste une option pour la compatibilité, et SCP est utile pour les transferts ponctuels simples.

L'avenir tend vers :

- Une généralisation du chiffrement TLS 1.3 et des clés post-quantiques.
- L'intégration native des transferts sécurisés dans les pipelines CI/CD.
- L'automatisation poussée avec des outils comme Rclone, Ansible et Terraform.
- Une supervision renforcée via SIEM et Zero Trust Network Access (ZTNA).

**Auteur :** `hackers_tchad`  
**Date :** 2026  
**Version :** 2.0  
**Objectif :** Référence éducative, opérationnelle et architecturale pour les professionnels du réseau et de la sécurité.

---

*Fin des annexes avancées.*
