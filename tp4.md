

# TP4 — Administration SSH et Serveur Web Nginx

Auteur : Nathan  
Environnement : Ubuntu Server (VM VirtualBox)

---

# Partie 1 — Mise en place de la VM

## Configuration VM

- 2 Go RAM
- 20 Go disque
- Réseau : Bridged Adapter

Vérification IP sur la VM :

```bash
ip a
```

Test depuis la machine hôte :

```bash
ping <IP_VM>
```

La VM est accessible depuis l’hôte.

---

# Partie 2 — Installation et configuration SSH

## Installation

```bash
sudo apt update
sudo apt install openssh-server -y
```

## Vérification service

```bash
sudo systemctl status ssh
ss -tulnp | grep ssh
```

Le service SSH écoute sur le port 22.

## Connexion depuis l’hôte

```bash
ssh etudiant@<IP_VM>
```

## Génération clé SSH côté client

```bash
ssh-keygen
```

Copie clé vers serveur :

```bash
ssh-copy-id etudiant@<IP_VM>
```

Connexion sans mot de passe validée.

---

# Partie 3 — Sécurisation SSH

Édition configuration :

```bash
sudo nano /etc/ssh/sshd_config
```

Modifications :

```
PermitRootLogin no
PasswordAuthentication no
Port 2222
```

Redémarrage SSH :

```bash
sudo systemctl restart ssh
```

Connexion avec nouveau port :

```bash
ssh -p 2222 etudiant@<IP_VM>
```

## Alias SSH

Sur machine cliente :

```bash
nano ~/.ssh/config
```

Ajout :

```
Host serveur-tp
    HostName <IP_VM>
    User etudiant
    Port 2222
```

Connexion simplifiée :

```bash
ssh serveur-tp
```

---

# Partie 4 — Transfert de fichiers

## SCP

```bash
scp fichier.txt serveur-tp:/home/etudiant/
```

## SFTP

```bash
sftp serveur-tp
put fichier.txt
ls
get fichier.txt
```

## RSYNC

```bash
rsync -avz dossier/ serveur-tp:/home/etudiant/dossier/
```

---

# Partie 5 — Logs et sécurité

## Logs SSH

```bash
sudo tail -f /var/log/auth.log
```

Observation des connexions et tentatives.

## Installation Fail2Ban

```bash
sudo apt install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

Vérification :

```bash
sudo fail2ban-client status
```

Fail2Ban bloque les IP après tentatives échouées répétées.

---

# Partie 6 — Tunnel SSH

## Tunnel local

```bash
ssh -L 8080:localhost:80 serveur-tp
```

Permet d’accéder au service web distant via :

```
http://localhost:8080
```

## Tunnel distant

```bash
ssh -R 2223:localhost:22 serveur-tp
```

Permet accès SSH au client via le serveur.

---

# Partie 7 — Installation Nginx + HTTPS

## Installation

```bash
sudo apt install nginx -y
```

## Création site

```bash
sudo mkdir -p /var/www/site-tp
sudo nano /var/www/site-tp/index.html
```

Contenu :

```html
<h1>Bienvenue sur le site TP4</h1>
```

## Configuration Nginx HTTP

```bash
sudo nano /etc/nginx/sites-available/site-tp
```

Configuration :

```
server {
    listen 80;
    server_name <IP_VM>;

    root /var/www/site-tp;
    index index.html;
}
```

Activation :

```bash
sudo ln -s /etc/nginx/sites-available/site-tp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## Certificat SSL auto-signé

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
-keyout /etc/ssl/private/site-tp.key \
-out /etc/ssl/certs/site-tp.crt
```

Ajout HTTPS + redirection :

```
server {
    listen 80;
    server_name <IP_VM>;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name <IP_VM>;

    ssl_certificate /etc/ssl/certs/site-tp.crt;
    ssl_certificate_key /etc/ssl/private/site-tp.key;

    root /var/www/site-tp;
    index index.html;
}
```

Redémarrage :

```bash
sudo nginx -t
sudo systemctl restart nginx
```

Test :

```bash
curl -k https://<IP_VM>
```

---

# Partie 8 — Firewall et permissions

## Activation UFW

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw status
```

## Permissions

```bash
sudo chown -R www-data:www-data /var/www/site-tp
sudo chmod -R 755 /var/www/site-tp
```

Nginx peut lire les fichiers correctement.

---

# Partie 9 — Validation finale

✔ SSH fonctionne sur port 2222  
✔ Authentification par clé uniquement  
✔ Root désactivé  
✔ Fail2Ban actif  
✔ SCP / SFTP / RSYNC fonctionnels  
✔ Nginx accessible en HTTP et HTTPS  
✔ Redirection HTTP → HTTPS active  
✔ Certificat auto-signé valide  
✔ Firewall configuré  
✔ Permissions correctes

---

# Conclusion

Ce TP a permis de :

- Mettre en place un serveur SSH sécurisé
- Implémenter une authentification par clé
- Sécuriser un service avec Fail2Ban
- Déployer un serveur web Nginx
- Configurer HTTPS avec certificat SSL
- Gérer firewall et permissions
- Comprendre les tunnels SSH

L’administration sécurisée d’un serveur Linux a été validée.

---