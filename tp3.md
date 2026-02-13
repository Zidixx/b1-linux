

# B1 Linux — TP3 Cryptographie (OpenSSL)

# I. Exploration — OpenSSL

## A. Base64

### 1. Génération d’un fichier binaire

Création d’un fichier binaire aléatoire de 100 Ko :

```bash
dd if=/dev/urandom of=data.bin bs=1k count=100
```

Vérification de la taille :

```bash
ls -lh data.bin
```

---

### 2. Encodage Base64

Encodage :

```bash
openssl base64 -e -in data.bin -out data.b64
```

Affichage :

```bash
cat data.b64
```

Comparaison des tailles :

```bash
ls -lh data.*
```

➡ Le fichier encodé est plus grand (~33%).

---

### 3. Décodage

```bash
openssl base64 -d -in data.b64 -out data_restored.bin
```

Vérification :

```bash
diff -s data.bin data_restored.bin
```

Résultat : fichiers identiques.

---

### 4. Réponses — Base64

- Base64 n’est **pas un chiffrement**, c’est un encodage texte.
- La taille augmente car 3 octets binaires deviennent 4 caractères.
- Augmentation ≈ **33%**.
- `diff` permet de vérifier rigoureusement l’identité.

---

## B. Chiffrement symétrique — AES

---

### 1. Création du message

```bash
nano confidentiel.txt
```

Contenu :

- nom
- date
- 5 lignes minimum

---

### 2. Chiffrement AES‑256

```bash
openssl enc -e -aes256 -salt -pbkdf2 -md sha256 \
-in confidentiel.txt -out confidentiel.enc
```

Vérification binaire :

```bash
cat confidentiel.enc
```

---

### 3. Déchiffrement

```bash
openssl enc -d -aes256 -pbkdf2 -md sha256 \
-in confidentiel.enc -out confidentiel_dechiffre.txt
```

Comparaison :

```bash
diff -s confidentiel.txt confidentiel_dechiffre.txt
```

---

### 4. Analyse

Deuxième chiffrement :

```bash
openssl enc -e -aes256 -salt -pbkdf2 -md sha256 \
-in confidentiel.txt -out confidentiel2.enc
```

Comparaison :

```bash
diff confidentiel.enc confidentiel2.enc
```

➡ fichiers différents.

---

### Réponses — AES

- Différence due au **sel aléatoire**.
- Le sel empêche les attaques par dictionnaire.
- Changer une option rend le déchiffrement impossible.
- PBKDF2 renforce la dérivation de clé.
- Encodage ≠ chiffrement (lisible vs protégé).

---

## C. Cryptographie asymétrique — RSA

---

### 1. Génération des clés

```bash
openssl genrsa -aes256 -out rsa_private.pem 2048
```

Clé publique :

```bash
openssl rsa -in rsa_private.pem -pubout -out rsa_public.pem
```

Paramètres :

```bash
openssl rsa -in rsa_private.pem -text -noout
openssl rsa -pubin -in rsa_public.pem -text -noout
```

Comparaison : la clé privée contient des paramètres secrets supplémentaires.

---

### 2. Chiffrement asymétrique

Création :

```bash
nano secret.txt
```

Chiffrement :

```bash
openssl pkeyutl -encrypt -in secret.txt \
-inkey rsa_public.pem -pubin -out secret.enc
```

Déchiffrement :

```bash
openssl pkeyutl -decrypt -in secret.enc \
-inkey rsa_private.pem -out secret_dechiffre.txt
```

---

### Réponses — RSA

- La clé privée doit rester secrète.
- RSA est lent pour gros fichiers.
- Clé privée = paramètres complets.
- Modulo = base mathématique RSA.
- RSA sert souvent à protéger une clé AES.

---

## D. Signature numérique

---

### 1. Création et signature

```bash
nano contrat.txt
```

Hash :

```bash
openssl dgst -sha256 contrat.txt
```

Signature :

```bash
openssl dgst -sha256 -sign rsa_private.pem \
-out contrat.sig contrat.txt
```

---

### 2. Vérification

```bash
openssl dgst -sha256 -verify rsa_public.pem \
-signature contrat.sig contrat.txt
```

Modifier le fichier puis revérifier → échec.

---

### Réponses — Signature

- Toute modification invalide la signature.
- Le hash garantit l’intégrité.
- Signature = authenticité.
- Chiffrement = confidentialité.

---

## Bonus — Chiffrement hybride

Clé AES :

```bash
openssl rand -base64 32 > aes.key
```

Chiffrement fichier :

```bash
openssl enc -aes256 -salt -pbkdf2 \
-in secret.txt -out secret_aes.enc -pass file:aes.key
```

Protection clé AES :

```bash
openssl pkeyutl -encrypt -in aes.key \
-inkey rsa_public.pem -pubin -out aes.key.enc
```

➡ Combine rapidité AES + sécurité RSA.

---

# Bilan

- Base64 = encodage
- AES = chiffrement symétrique rapide
- RSA = chiffrement asymétrique sécurisé
- Signature = intégrité + authenticité
- OpenSSL permet toutes ces opérations en CLI

---