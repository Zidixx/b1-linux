

# TP6 – Mise en place d’un serveur OpenVPN sur Ubuntu Server

---

# Objectifs

- Installer et configurer OpenVPN sur Ubuntu Server
- Mettre en place une PKI (Infrastructure de certificats)
- Comprendre le NAT et le routage IP
- Générer un profil client fonctionnel
- Diagnostiquer un service système

---

# 1. Préparation du système

## Mise à jour

```bash
sudo apt update
sudo apt upgrade -y
```

## Installation

```bash
sudo apt install openvpn easy-rsa -y
```

---

# 2. Infrastructure PKI (Easy-RSA)

## Création de l’environnement

```bash
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
./easyrsa init-pki
```

## Génération de la CA

```bash
./easyrsa build-ca
```

## Certificat serveur

```bash
./easyrsa gen-req server nopass
./easyrsa sign-req server server
```

## Certificat client

```bash
./easyrsa gen-req client1 nopass
./easyrsa sign-req client client1
```

## Paramètres Diffie-Hellman

```bash
./easyrsa gen-dh
```

## Clé TLS

```bash
openvpn --genkey --secret ta.key
```

---

# Questions – PKI

### À quoi sert une CA ?
Elle signe les certificats et garantit l’authenticité des entités.

### Différence clé privée / certificat ?
La clé privée reste secrète.  
Le certificat contient la clé publique signée par la CA.

### Pourquoi un serveur VPN a besoin de certificats ?
Pour authentifier le serveur et les clients et sécuriser l’échange de clés.

### Où Easy-RSA crée les fichiers ?
Dans le dossier `pki/`.

### Que contient pki/ ?
- ca.crt
- private/
- issued/
- dh.pem
- reqs/

### Différence gen-req / sign-req ?
- gen-req : génère clé + demande de signature  
- sign-req : signe la demande avec la CA

### Si on oublie de signer ?
Le certificat ne sera pas valide → connexion refusée.

---

# 3. Configuration OpenVPN

## Dossier serveur

```bash
sudo mkdir -p /etc/openvpn/server
```

## Copie des fichiers

```bash
sudo cp pki/ca.crt /etc/openvpn/server/
sudo cp pki/private/server.key /etc/openvpn/server/
sudo cp pki/issued/server.crt /etc/openvpn/server/
sudo cp pki/dh.pem /etc/openvpn/server/
sudo cp ta.key /etc/openvpn/server/
```

## Fichier server.conf

`/etc/openvpn/server/server.conf`

```
port 1194
proto udp
dev tun

ca ca.crt
cert server.crt
key server.key
dh dh.pem
tls-auth ta.key 0

server 10.8.0.0 255.255.255.0

push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"

cipher AES-256-CBC
auth SHA256

user nobody
group nogroup
persist-key
persist-tun

verb 3
```

---

# Questions – Configuration

### Que signifie dev tun ?
Interface virtuelle de niveau 3 (IP).

### UDP vs TCP ?
UDP est plus rapide et adapté aux VPN.  
TCP est plus fiable mais plus lourd.

### Plage IP VPN ?
10.8.0.0/24 (réseau privé non utilisé ailleurs).

---

# 4. Routage et NAT

## Activer IP forwarding

Dans `/etc/sysctl.conf` :

```
net.ipv4.ip_forward=1
```

Puis :

```bash
sudo sysctl -p
```

## NAT (Masquerade)

```bash
sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o enp0s3 -j MASQUERADE
```

---

# Questions – NAT

### Où configurer ip_forward ?
Dans `/etc/sysctl.conf`.

### Voir règles NAT ?
```bash
sudo iptables -t nat -L -v
```

### Pourquoi masquerader ?
Pour que le trafic VPN sorte avec l’IP publique du serveur.

---

# 5. Démarrage du service

```bash
sudo systemctl start openvpn-server@server
sudo systemctl status openvpn-server@server
```

## Logs en cas d’erreur

```bash
journalctl -xeu openvpn-server@server
```

---

# Questions – Service

### status vs journalctl ?
- status : état rapide  
- journalctl : logs détaillés

---

# 6. Profil client (.ovpn)

```
client
dev tun
proto udp
remote IP_SERVEUR 1194

persist-key
persist-tun

cipher AES-256-CBC
auth SHA256
key-direction 1

<ca>
(contenu ca.crt)
</ca>

<cert>
(contenu client1.crt)
</cert>

<key>
(contenu client1.key)
</key>

<tls-auth>
(contenu ta.key)
</tls-auth>
```

---

# Questions – Client

### Intégrer certificat dans .ovpn ?
Avec balises `<ca>`, `<cert>`, `<key>`.

### Pourquoi ne jamais partager la clé privée ?
Elle permet d’usurper l’identité du client.

---

# 7. Tests

## Vérifier interface VPN

```bash
ip a
```

Interface attendue : `tun0`

## Vérifier IP publique

```bash
curl ifconfig.me
```

---

# Questions – Validation

### Comment vérifier que le trafic passe par le VPN ?
Comparer l’IP publique avant/après connexion.

### Si port 1194 bloqué ?
Connexion impossible → timeout.

---

# Bonus

- Ajouter clients supplémentaires
- Révoquer certificat : `./easyrsa revoke client1`
- Passer en TCP : `proto tcp`
- Ajouter authentification par mot de passe

---

# Conclusion

OpenVPN configuré avec :

- PKI complète
- NAT fonctionnel
- Routage IP activé
- Profil client opérationnel
- Service vérifié via systemctl et journalctl

TP validé.