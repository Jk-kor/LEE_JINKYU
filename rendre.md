### Généralement, lors de la configuration de la sécurité, il peut arriver que les fichiers d'installation nécessaires manquent si Linux est nouvellement installé. Pour les installer rapidement et commencer..
```bash
sudo apt update
sudo apt upgrade
sudo apt install build-essential
sudo apt install gcc unzip curl perl python3-pip git
sudo apt install xtables-addons-common iptables-persistent libnet-cidr-lite-perl libtext-csv-perl
```

### Mise à jour du système et installation des packages nécessaires.
```bash
sudo apt update
sudo apt upgrade
sudo apt install build-essential
sudo apt install gcc unzip curl perl python3-pip git
sudo apt install xtables-addons-common iptables-persistent libnet-cidr-lite-perl libtext-csv-perl
```

### Configuration de la sécurité SSH
```bash
sudo vim /etc/ssh/sshd_config
PermitRootLogin no
StrictModes yes
MaxAuthTries 2
MaxSessions 2
```

### Vérification des journaux de connexion
```bash
sudo last -f /var/log/btmp # Vérifier les échecs de connexion
sudo last -f /var/log/wtmp # Vérifier les succès de connexion
```

### Suppression des comptes par défaut inutilisés
```bash
cat /etc/passwd | egrep "lp|uucp|nuucp"
sudo userdel lp
sudo userdel uucp
```

### Configuration de la politique de mot de passe
#### Cela signifie que le mot de passe doit comporter au moins 9 caractères, être changé entre 7 et 90 jours, et un avertissement sera affiché 10 jours avant l'expiration des 90 jours. Si vous ne prévoyez pas de changer régulièrement le mot de passe, modifiez uniquement PASS_MIN_LEN.
```bash
sudo apt reinstall passwd
sudo vim /etc/login.defs
PASS_MIN_LEN 9
PASS_MAX_DAYS 90
PASS_MIN_DAYS 7
PASS_WARN_AGE 10
```

### Ajout d'un utilisateur
```bash
sudo groupadd jin
sudo adduser jinroot
sudo passwd jinroot 
```

### Configuration des permissions : Pour permettre à tous les membres du groupe jin d'utiliser la commande sudo pour obtenir les droits root, modifiez le fichier /etc/sudoers
```bash
sudo visudo
%jin ALL=(ALL) NOPASSWD:ALL
```

### Configuration du proxy inverse Nginx
```bash
user www-data;                          # Utilisateur Nginx
worker_processes auto;                   # Nombre optimal de processus
pid /run/nginx.pid;                      # Fichier PID
error_log /var/log/nginx/error.log;      # Fichier de log des erreurs

include /etc/nginx/modules-enabled/*.conf;  # Inclut les modules activés

events {
    worker_connections 768;             # Connexions simultanées maximales
}

http {
    include /etc/nginx/mime.types;      # Types MIME
    default_type application/octet-stream;  # Type MIME par défaut
    types_hash_max_size 2048;           # Taille maximale de la table de hash des types MIME

    limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;  # Limitation des requêtes réseau

    sendfile on;                         # Transfert direct des fichiers
    tcp_nopush on;                       # Désactive la réduction TCP_NOPUSH

    server_names_hash_bucket_size 128;   # Taille de la table de hash des noms de serveur

    gzip on;                             # Compression gzip
    gzip_vary on;                        # Modifie la réponse pour les clients qui acceptent gzip
    gzip_proxied any;                    # Compression gzip pour toutes les proxys
    gzip_comp_level 6;                   # Niveau de compression gzip
    gzip_buffers 16 8k;                  # Taille des buffers de compression
    gzip_http_version 1.1;               # Version HTTP pour la compression
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    server {
        listen 80;                       # Écoute sur le port 80
        server_name 192.168.1.146;        # Nom du serveur

        location / {                      # Redirection des requêtes
            allow 192.168.1.100;          # Autorise IP spécifique
            allow 192.168.1.146;          # Autorise IP spécifique
            deny all;                     # Interdit autres IP
            proxy_pass http://127.0.0.1:8080;  # Redirige au backend
            proxy_set_header Host $host;   # Définit l'en-tête Host pour le proxy
            proxy_set_header X-Real-IP $remote_addr;  # Définit l'IP réelle du client

            # Limitation du taux de requêtes
            limit_req zone=one burst=5;   # Limite à 10 req/s avec pic de 5
        }

        # Configuration SSL
        ssl_protocols TLSv1.2 TLSv1.3;   # Protocoles SSL/TLS
        ssl_prefer_server_ciphers on;     # Préfère les ciphers pour le serveur SSL
        
        # Journalisation d'accès
        access_log /var/log/nginx/access.log;  # Journalisation des accès
    }
}

```

## Vérification des échecs de connexion
```bash
sudo last -f /var/log/btmp 
```

## Vérification des succès de connexion
```bash
sudo last -f /var/log/wtmp 
```

## Pour voir les gens qui ont essayé de se connecter
```bash
nano check_log.sh
#!/bin/bash

echo "### Échecs de connexion ###"
sudo last -f /var/log/btmp
echo ""
echo "### Succès de connexion ###"
sudo last -f /var/log/wtmp
```

## Attribution des droits d'exécution au script
```bash
chmod +x check_log.sh
```

## Configurer un pare-feu avec firewalld
```bash
sudo apt install firewalld
sudo systemctl enable firewalld
sudo systemctl start firewalld
```

## Installation et configuration de fail2ban
```bash
sudo dnf install epel-release -y
sudo dnf install fail2ban -y
```
```bash
sudo nano /etc/fail2ban/jail.local
[sshd]
enabled = true
logpath = /var/log/secure
maxretry = 3
bantime = 600
```
```bash
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```
