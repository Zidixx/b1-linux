# TP5 – pfSense : Bases d’un pare-feu

---

# Partie 1 – Prise en main et sécurisation

## 1. Accès à l’interface

📷 `01_interfaces_overview.png`

### Quelle est l’adresse IP du LAN ?
192.168.128.2/24

### Quelle est l’adresse IP du WAN ?
192.168.64.9

### Pourquoi utilise-t-on HTTPS ?
Pour chiffrer les communications d’administration et éviter l’interception des identifiants.

### Pourquoi faut-il changer les identifiants par défaut ?
Les identifiants par défaut sont connus publiquement → risque de compromission immédiate.

---

## 2. Sécurisation de l’accès administrateur

📷 `02_system_users_admin.png`

### Où se gèrent les utilisateurs ?
System → User Manager

### Qu’est-ce qu’un mot de passe robuste ?
- Long (>12 caractères)
- Majuscules / minuscules
- Chiffres
- Caractères spéciaux
- Non présent dans un dictionnaire

### Pourquoi sécuriser en priorité l’accès admin ?
Car le pare-feu contrôle tout le trafic réseau → compromission = perte totale de sécurité.

---

# Partie 2 – Interfaces réseau

## 3. Vérification des interfaces

📷 `03_interface_assignments.png`  
📷 `04_wan_configuration_dhcp.png`  
📷 `05_lan_configuration_static_ipv4.png`

### Quelle interface permet l’accès Internet ?
WAN

### Quelle interface correspond au réseau interne ?
LAN

### Que se passerait-il si elles étaient inversées ?
Le LAN serait exposé directement à Internet → faille critique.

---

# Partie 3 – Services réseau

## 4. DHCP

📷 `08_lan_dhcp_configuration.png`

### Pourquoi utiliser DHCP ?
Distribution automatique des IP → gestion simplifiée.

### Quelle plage choisir ?
Exemple : 192.168.128.11 – 192.168.128.245

### Quelles adresses éviter ?
- IP du pare-feu (192.168.128.2)
- IP statiques (serveurs)

### Vérification Ubuntu

Commande :

```bash
ip a
```

📷 `06_client_ip_configuration.png`

Ubuntu obtient bien automatiquement une IP.

---

## 5. DNS

### Pourquoi le pare-feu peut jouer le rôle de DNS ?
Centralisation des requêtes + contrôle + filtrage possible.

### Si DNS ne fonctionne pas mais ping 8.8.8.8 fonctionne ?
Le réseau fonctionne, mais la résolution de noms échoue.

Test :

```bash
ping 8.8.8.8
nslookup google.com
```

---

# Partie 4 – Autoriser Internet

## 6. Règles pare-feu

📷 `12_firewall_lan_rules_blocking.png`

### Source ?
LAN net

### Destination ?
Any

### Autoriser tous les protocoles ?
Oui pour accès Internet général (IPv4 *)

### Tests

```bash
ping 192.168.128.2
ping 8.8.8.8
nslookup google.com
curl http://google.com
```

📷 `07_lan_client_ping_internet_success.png`

### Si ça ne fonctionne pas ?
Vérifier :
- Firewall → Rules
- Firewall → Logs
- NAT
- Gateway

---

## 7. NAT

### Pourquoi NAT est nécessaire ?
Car WAN est en réseau NAT (VirtualBox).  
Il faut traduire les IP privées vers IP WAN.

### Différence NAT automatique / manuel ?
- Automatique : pfSense crée les règles
- Manuel : configuration personnalisée

### Vérification traduction ?
Firewall → Diagnostics → States  
Ou consulter les logs.

---

# Partie 5 – Filtrage

## 8. Blocage site spécifique

📷 `09_firewall_alias_site_bloque.png`  
📷 `10_site_bloque_test_curl_failed.png`

### Bloquer par IP ou domaine ?
Par IP ou via DNS Resolver.

### Si HTTPS ?
Impossible d’inspecter contenu sans inspection SSL.

### Pourquoi blocage IP contournable ?
- CDN
- Changement IP
- VPN

Logs visibles dans Firewall → Logs.

---

## 9. Blocage catégorie (jeux d’argent)

📷 `11_firewall_aliases_blocked_sites.png`

### Pourquoi pas une règle par site ?
Non maintenable → utiliser alias.

### Où créer alias ?
Firewall → Aliases

### Vérification blocage ?
Test :

```bash
curl http://betclic.fr
```

📷 `13_block_test_terminal_betclic.png`

---

# Partie 6 – Aller plus loin

## 10. Blocage réseaux sociaux

Créer alias + règle.

### Si règle sous "Pass Any" ?
Elle ne sera jamais appliquée (ordre top → bottom).

---

## 11. Règles horaires

Créer schedule → appliquer à règle.

### Pourquoi utile en entreprise ?
Limiter accès selon horaires de travail.

---

## 12. Serveur web local

Installation Ubuntu :

```bash
sudo apt update
sudo apt install apache2
```

### Filtrer par IP source ?
Oui pour limiter accès.

### Filtrer par port ?
Oui (autoriser uniquement 80/443).

### Pourquoi pare-feu protège LAN ?
Car il contrôle tout trafic entrant/sortant.

---

## 13. Logs

Activer "Log packets handled".

### Différence paquet bloqué / autorisé ?
- Pass = autorisé
- Block = refusé

### Identifier règle ?
Colonne "Rule" dans logs.

### Sens trafic ?
Source → Destination.

---

## 14. DMZ

### Qu’est-ce qu’une DMZ ?
Réseau isolé pour serveurs exposés.

### Pourquoi isoler ?
Limiter impact en cas de compromission.

### Machine DMZ vers LAN ?
Non (règles restrictives).

### LAN vers DMZ ?
Oui si autorisé explicitement.

---

## 15. Filtrage MAC

### Est-ce sécurisé ?
Non.

### Pourquoi contournable ?
Adresse MAC falsifiable (spoofing).

---

## 16. Portail captif

### Contextes ?
Wi-Fi public, hôtels, entreprises.

### Avantage vs simple règle ?
Authentification + traçabilité utilisateur.

---

## 17. Sauvegarde / restauration

System → Backup & Restore

### Pourquoi sauvegarde essentielle ?
Permet restauration rapide en cas d’erreur ou panne.

---

# Conclusion

Objectifs atteints :
- Installation pfSense
- DHCP / DNS
- NAT
- Règles pare-feu
- Alias
- Filtrage ciblé
- Tests fonctionnels validés

# TP5 – pfSense : Bases d’un pare-feu

---

# Partie 1 – Prise en main et sécurisation

## 1. Accès à l’interface

📷 `01_interfaces_overview.png`

### Quelle est l’adresse IP du LAN ?
192.168.128.2/24  
C’est l’adresse interne du pare-feu utilisée comme passerelle par les machines du réseau local.

### Quelle est l’adresse IP du WAN ?
192.168.64.9  
C’est l’adresse obtenue via DHCP sur le réseau externe (VirtualBox NAT).

### Pourquoi utilise-t-on HTTPS ?
HTTPS chiffre les échanges entre l’administrateur et le pare-feu.  
Cela empêche l’interception des identifiants ou des données sensibles sur le réseau.

### Pourquoi faut-il changer les identifiants par défaut ?
Les identifiants par défaut sont publics et connus.  
Les laisser actifs expose immédiatement l’équipement à une compromission.

---

## 2. Sécurisation de l’accès administrateur

📷 `02_system_users_admin.png`

### Où se gèrent les utilisateurs ?
System → User Manager

### Qu’est-ce qu’un mot de passe robuste ?
Un mot de passe robuste :
- Long (>12 caractères)
- Mélange majuscules, minuscules, chiffres, symboles
- Non basé sur un mot simple ou personnel

### Pourquoi sécuriser en priorité l’accès admin ?
Le pare-feu contrôle l’intégralité du trafic réseau.  
Un accès administrateur compromis permettrait de modifier les règles, espionner le trafic ou ouvrir le réseau.

---

# Partie 2 – Interfaces réseau

## 3. Vérification des interfaces

📷 `03_interface_assignments.png`  
📷 `04_wan_configuration_dhcp.png`  
📷 `05_lan_configuration_static_ipv4.png`

### Quelle interface permet l’accès Internet ?
L’interface WAN.  
Elle est connectée au réseau externe et sert de sortie vers Internet.

### Quelle interface correspond au réseau interne ?
L’interface LAN.  
Elle distribue les services (DHCP, DNS) aux machines internes.

### Que se passerait-il si elles étaient inversées ?
Le réseau interne serait exposé directement à Internet sans protection, ce qui représenterait une faille critique de sécurité.

---

# Partie 3 – Services réseau

## 4. DHCP

📷 `08_lan_dhcp_configuration.png`

### Pourquoi utiliser DHCP ?
DHCP permet d’attribuer automatiquement les adresses IP, ce qui simplifie la gestion du réseau et évite les conflits d’adresses.

### Quelle plage choisir ?
Une plage cohérente avec le réseau LAN, par exemple :  
192.168.128.11 – 192.168.128.245

### Quelles adresses éviter ?
- L’adresse du pare-feu (192.168.128.2)
- Les adresses réservées aux serveurs ou machines statiques

### Vérification Ubuntu

```bash
ip a
```

📷 `06_client_ip_configuration.png`

Ubuntu obtient automatiquement une adresse IP via le serveur DHCP.

---

## 5. DNS

### Pourquoi le pare-feu peut jouer le rôle de DNS ?
Il centralise les requêtes DNS, permet le filtrage (blocage de domaines) et améliore le contrôle du trafic.

### Si DNS ne fonctionne pas mais ping 8.8.8.8 fonctionne ?
La connectivité IP fonctionne, mais la résolution de noms échoue.  
Le problème est donc lié au service DNS et non au routage.

Tests :

```bash
ping 8.8.8.8
nslookup google.com
```

---

# Partie 4 – Autoriser Internet

## 6. Règles pare-feu

📷 `12_firewall_lan_rules_blocking.png`

### Source ?
LAN net  
Toutes les machines internes.

### Destination ?
Any  
Vers toutes les destinations externes.

### Autoriser tous les protocoles ?
Oui dans un contexte de base, pour permettre HTTP, HTTPS, DNS, ICMP, etc.

### Tests

```bash
ping 192.168.128.2
ping 8.8.8.8
nslookup google.com
curl http://google.com
```

📷 `07_lan_client_ping_internet_success.png`

### Si ça ne fonctionne pas ?
Vérifier :
- Firewall → Rules (ordre des règles)
- Firewall → Logs
- NAT
- Gateway

---

## 7. NAT

### Pourquoi NAT est nécessaire ?
Les adresses privées (192.168.x.x) ne sont pas routables sur Internet.  
Le NAT traduit ces adresses vers l’adresse WAN.

### Différence NAT automatique / manuel ?
- Automatique : pfSense génère les règles automatiquement.
- Manuel : l’administrateur définit précisément les traductions.

### Vérification traduction ?
Diagnostics → States  
Ou consultation des logs firewall.

---

# Partie 5 – Filtrage

## 8. Blocage site spécifique

📷 `09_firewall_alias_site_bloque.png`  
📷 `10_site_bloque_test_curl_failed.png`

### Bloquer par IP ou domaine ?
On peut bloquer par IP ou via DNS (plus flexible).

### Si HTTPS ?
Le contenu ne peut pas être inspecté sans inspection SSL, mais la connexion peut être bloquée.

### Pourquoi blocage IP contournable ?
- Les sites utilisent plusieurs IP (CDN)
- Les IP changent
- Utilisation possible d’un VPN

Les logs permettent de vérifier que le trafic est bien bloqué.

---

## 9. Blocage catégorie (jeux d’argent)

📷 `11_firewall_aliases_blocked_sites.png`

### Pourquoi pas une règle par site ?
Ce serait difficile à maintenir.  
Un alias permet de regrouper plusieurs domaines dans une seule règle.

### Où créer alias ?
Firewall → Aliases

### Vérification blocage ?

```bash
curl http://betclic.fr
```

📷 `13_block_test_terminal_betclic.png`

---

# Partie 6 – Aller plus loin

## 10. Blocage réseaux sociaux

Création d’un alias regroupant plusieurs domaines (facebook.com, instagram.com, etc.)  
Application d’une règle de blocage au-dessus du "Pass Any".

### Si règle sous "Pass Any" ?
Elle ne sera jamais exécutée car pfSense applique les règles de haut en bas.

---

## 11. Règles horaires

Création d’un schedule puis association à une règle.

### Pourquoi utile en entreprise ?
Permet de restreindre certains usages (réseaux sociaux, streaming) uniquement pendant les heures de travail.

---

## 12. Serveur web local

Installation :

```bash
sudo apt update
sudo apt install apache2
```

### Filtrer par IP source ?
Oui, pour autoriser uniquement certaines machines.

### Filtrer par port ?
Oui, en autorisant uniquement le port 80 ou 443.

### Pourquoi pare-feu protège LAN ?
Il filtre tout trafic entre interfaces, même interne.

---

## 13. Logs

Activation de la journalisation sur les règles.

### Différence paquet bloqué / autorisé ?
- Pass : trafic autorisé
- Block : trafic refusé

### Identifier règle ?
La colonne "Rule" dans les logs indique quelle règle a été appliquée.

### Sens trafic ?
Source → Destination.

---

## 14. DMZ

### Qu’est-ce qu’une DMZ ?
Un réseau séparé destiné aux serveurs exposés à Internet.

### Pourquoi isoler ?
Limiter l’impact en cas de compromission.

### Machine DMZ vers LAN ?
Non, sauf règle spécifique.

### LAN vers DMZ ?
Oui si explicitement autorisé.

---

## 15. Filtrage MAC

### Est-ce sécurisé ?
Non.

### Pourquoi contournable ?
Une adresse MAC peut être modifiée (spoofing).

---

## 16. Portail captif

### Contextes ?
Réseaux Wi-Fi publics, hôtels, entreprises.

### Avantage ?
Authentification des utilisateurs et traçabilité.

---

## 17. Sauvegarde / restauration

System → Backup & Restore

### Pourquoi sauvegarde essentielle ?
Permet une restauration rapide après erreur de configuration ou panne matérielle.

---

# Conclusion

Objectifs atteints :
- Installation et configuration pfSense
- Mise en place DHCP et DNS
- Configuration NAT
- Création règles firewall
- Mise en place alias
- Filtrage ciblé
- Tests fonctionnels validés