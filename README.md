# 🔏 signtool

> **Outil CLI de signature numérique de fichiers** basé sur RSA + SHA-256 (PKCS#1 v1.5).  
> Signez, vérifiez et inspectez l'authenticité de n'importe quel fichier depuis le terminal.

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![Python](https://img.shields.io/badge/python-%3E%3D3.8-brightgreen)
![Licence](https://img.shields.io/badge/licence-MIT-green)
![Plateforme](https://img.shields.io/badge/plateforme-Windows%20%7C%20Linux%20%7C%20macOS-lightgrey)

---

## Sommaire

1. [Présentation](#présentation)
2. [Fonctionnalités](#fonctionnalités)
3. [Installation](#installation)
4. [Démarrage rapide](#démarrage-rapide)
5. [Commandes](#commandes)
   - [keygen](#-signtool-keygen----générer-une-paire-de-clés-rsa)
   - [sign](#-signtool-sign----signer-un-ou-plusieurs-fichiers)
   - [verify](#-signtool-verify----vérifier-une-signature)
   - [info](#-signtool-info----inspecter-un-fichier-sig)
6. [Format du fichier .sig](#format-du-fichier-sig)
7. [Workflow complet](#workflow-complet)
8. [Démos](#démos)
9. [Structure du projet](#structure-du-projet)
10. [Sécurité](#sécurité)
11. [Dépendances](#dépendances)
12. [Licence](#licence)

---

## Présentation

`signtool` est un outil en ligne de commande qui permet de **signer numériquement** des fichiers à l'aide d'une clé privée RSA et de **vérifier** leur authenticité avec la clé publique correspondante.

Chaque signature produit un fichier `.sig` au format JSON contenant :
- la signature cryptographique (base64),
- le hash SHA-256 du fichier original,
- l'algorithme utilisé,
- l'horodatage de signature,
- et les métadonnées de l'outil.

Cela garantit à la fois **l'authenticité** (le fichier provient bien du signataire) et **l'intégrité** (le fichier n'a pas été modifié).

---

## Fonctionnalités

| Fonctionnalité | Détail |
|---|---|
| 🔑 Algorithme | RSA (1024 / 2048 / 4096 bits) + SHA-256, padding PKCS#1 v1.5 |
| 📄 Format des clés | PEM (compatible OpenSSL) |
| 📦 Fichier de signature | `.sig` — JSON avec signature base64, algorithme, timestamp, hash SHA-256 |
| 🔒 Protection de la clé | Chiffrement AES optionnel par passphrase |
| 🗂️ Signature en masse | Patterns glob (`docs/*.pdf`) |
| 🎨 Sortie enrichie | Tableaux colorés, spinners, panneaux via `rich` |
| 🚦 Codes de retour | `0` = succès / signature valide, `1` = erreur / signature invalide |
| 🖥️ Multiplateforme | Windows, Linux, macOS |

---

## Installation

### Prérequis

- Python **>= 3.8**
- pip

### Depuis les sources (recommandé)

```bash
git clone <url-du-dépôt>
cd signtool
pip install -e .
```

La commande `signtool` est alors disponible globalement dans votre environnement Python.

### Dépendances uniquement

```bash
pip install -r requirements.txt
```

---

## Démarrage rapide

```bash
# 1. Générer une paire de clés RSA-2048
signtool keygen --name maclef

# 2. Signer un fichier
signtool sign --key maclef_private.pem --file rapport.pdf

# 3. Vérifier la signature
signtool verify --key maclef_public.pem --file rapport.pdf

# 4. Inspecter le fichier .sig
signtool info rapport.pdf.sig
```

---

## Commandes

### 🔑 `signtool keygen` — Générer une paire de clés RSA

```
signtool keygen [OPTIONS]
```

Génère une paire de clés RSA (clé privée + clé publique) et les sauvegarde en format PEM.

#### Options

| Option | Défaut | Description |
|---|---|---|
| `--bits` | `2048` | Taille de la clé : `1024`, `2048` ou `4096` bits |
| `--output-dir`, `-o` | `.` | Répertoire de sortie pour les fichiers PEM |
| `--name`, `-n` | `key` | Nom de base des fichiers générés |
| `--passphrase`, `-p` | *(aucune)* | Passphrase pour chiffrer la clé privée (AES) |

#### Fichiers générés

| Fichier | Usage |
|---|---|
| `{name}_private.pem` | **Clé privée** — à conserver secrète (permissions `600` sur Linux/macOS) |
| `{name}_public.pem` | **Clé publique** — à distribuer librement aux vérificateurs |

#### Exemples

```bash
# Paire de clés 2048 bits avec le nom par défaut
signtool keygen

# Paire de clés 4096 bits dans un répertoire dédié
signtool keygen --bits 4096 --output-dir ~/mes-clefs --name projet

# Clé privée protégée par une passphrase
signtool keygen --name maclef --passphrase "MonMotDePasse123"
```

---

### ✍️ `signtool sign` — Signer un ou plusieurs fichiers

```
signtool sign --key CLE_PRIVEE --file FICHIER [OPTIONS]
```

Signe un ou plusieurs fichiers avec la clé privée RSA. Chaque fichier `foo.txt` signé produit un fichier `foo.txt.sig`.

#### Options

| Option | Requis | Description |
|---|---|---|
| `--key`, `-k` | Oui | Chemin vers la clé privée PEM |
| `--file`, `-f` | Oui (répétable) | Fichier(s) à signer — supports les patterns glob et l'option multiple |
| `--output-dir`, `-o` | Non | Répertoire pour les fichiers `.sig` (défaut : même dossier que le fichier) |
| `--passphrase`, `-p` | Non | Passphrase si la clé privée est chiffrée |

#### Exemples

```bash
# Signer un fichier unique
signtool sign --key maclef_private.pem --file rapport.pdf

# Signer plusieurs fichiers explicitement
signtool sign --key maclef_private.pem \
              --file doc1.pdf \
              --file doc2.pdf \
              --file contrat.docx

# Signer tous les PDF d'un dossier (pattern glob)
signtool sign --key maclef_private.pem --file "docs/*.pdf"

# Stocker les .sig dans un répertoire séparé
signtool sign --key maclef_private.pem \
              --file "dist/*.tar.gz" \
              --output-dir signatures/

# Signer avec une clé protégée par passphrase
signtool sign --key maclef_private.pem \
              --file important.pdf \
              --passphrase "MonMotDePasse123"
```

---

### ✅ `signtool verify` — Vérifier une signature

```
signtool verify --key CLE_PUBLIQUE --file FICHIER [--sig FICHIER_SIG]
```

Vérifie l'authenticité et l'intégrité d'un fichier en deux étapes :
1. **Hash SHA-256** — le fichier n'a pas été modifié.
2. **Signature RSA** — la signature correspond bien à la clé publique.

#### Options

| Option | Requis | Description |
|---|---|---|
| `--key`, `-k` | Oui | Chemin vers la clé publique PEM |
| `--file`, `-f` | Oui | Fichier à vérifier |
| `--sig`, `-s` | Non | Chemin vers le `.sig` (défaut : `{fichier}.sig` dans le même dossier) |

#### Codes de retour

| Code | Signification |
|---|---|
| `0` | ✅ Signature **valide** — le fichier est authentique et intact |
| `1` | ❌ Signature **invalide** — fichier falsifié ou mauvaise clé |

#### Exemples

```bash
# Vérification simple (cherche automatiquement rapport.pdf.sig)
signtool verify --key maclef_public.pem --file rapport.pdf

# Spécifier manuellement le fichier .sig
signtool verify --key maclef_public.pem \
                --file rapport.pdf \
                --sig  signatures/rapport.pdf.sig

# Utilisation dans un script shell
if signtool verify --key pub.pem --file archive.zip; then
    echo "Archive authentique — installation autorisée."
else
    echo "ALERTE : le fichier a été falsifié !"
    exit 1
fi
```

---

### 🔍 `signtool info` — Inspecter un fichier .sig

```
signtool info FICHIER_SIG
```

Affiche toutes les métadonnées contenues dans un fichier `.sig` dans un panneau formaté.

#### Argument

| Argument | Description |
|---|---|
| `SIG_FILE` | Chemin vers le fichier `.sig` à inspecter |

#### Exemple

```bash
signtool info rapport.pdf.sig
```

**Sortie :**

```
 ╭─ Signature Metadata — rapport.pdf.sig ──────────────────────────────╮
 │ ╭──────────────┬──────────────────────────────────────────────────╮ │
 │ │ Field        │ Value                                            │ │
 │ ├──────────────┼──────────────────────────────────────────────────┤ │
 │ │ Sig file     │ /home/user/rapport.pdf.sig                       │ │
 │ │ Format ver   │ 1                                                │ │
 │ │ Tool         │ signtool v1.0.0                                  │ │
 │ │ Algorithm    │ RSA-SHA256-PKCS1v15                              │ │
 │ │ Filename     │ rapport.pdf                                      │ │
 │ │ Signed at    │ 2024-06-15T14:32:11.045782+00:00                 │ │
 │ │ SHA-256      │ e3b0c44298fc1c149afbf4c8996fb924...              │ │
 │ │ Signature    │ AbCdEf1234...                                    │ │
 │ │ Sig length   │ 344 chars (base64)                               │ │
 │ ╰──────────────┴──────────────────────────────────────────────────╯ │
 ╰──────────────────────────────────────────────────────────────────────╯
```

---

## Format du fichier .sig

Chaque signature est un fichier JSON lisible par l'humain :

```json
{
  "version": 1,
  "tool": "signtool v1.0.0",
  "algorithm": "RSA-SHA256-PKCS1v15",
  "timestamp": "2024-06-15T14:32:11.045782+00:00",
  "filename": "rapport.pdf",
  "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "signature": "AbCdEf1234...base64encodedRSAsignature...XyZ="
}
```

| Champ | Type | Description |
|---|---|---|
| `version` | entier | Version du format (pour la rétrocompatibilité) |
| `tool` | chaîne | Nom et version de l'outil ayant généré la signature |
| `algorithm` | chaîne | Algorithme utilisé (`RSA-SHA256-PKCS1v15`) |
| `timestamp` | chaîne | Date et heure de signature (ISO 8601, UTC) |
| `filename` | chaîne | Nom du fichier signé |
| `sha256` | chaîne | Hash SHA-256 hexadécimal du fichier original |
| `signature` | chaîne | Signature RSA encodée en Base64 |

---

## Workflow complet

Voici un exemple de bout en bout pour distribuer et vérifier des artefacts de release :

```bash
# Étape 1 — Le développeur génère ses clés (une seule fois)
signtool keygen --bits 2048 --output-dir ./clefs --name release

# Étape 2 — Il signe les artefacts de la release
signtool sign --key clefs/release_private.pem \
              --file "dist/*.tar.gz" \
              --output-dir dist/signatures/

# Étape 3 — Il publie :
#   - les artefacts (dist/*.tar.gz)
#   - les signatures (dist/signatures/*.sig)
#   - la clé publique (clefs/release_public.pem)

# --- Côté utilisateur ---

# Étape 4 — L'utilisateur vérifie le fichier téléchargé
signtool verify --key release_public.pem \
                --file monapp-1.0.tar.gz \
                --sig  signatures/monapp-1.0.tar.gz.sig

# Étape 5 — Il inspecte les détails de la signature si besoin
signtool info signatures/monapp-1.0.tar.gz.sig
```

---

## Démos

Des scripts de démonstration complets sont inclus pour tester l'ensemble du workflow en une seule commande.

**Windows :**

```bat
demo.bat
```

**Linux / macOS / WSL :**

```bash
chmod +x demo.sh
./demo.sh
```

La démo effectue les 7 étapes suivantes :

| Étape | Action |
|---|---|
| 1 | Installation des dépendances |
| 2 | Création de fichiers texte d'exemple |
| 3 | Génération d'une paire de clés RSA-2048 |
| 4 | Génération d'une paire de clés RSA-4096 |
| 5 | Signature d'un fichier unique, puis en lot |
| 6 | Vérification d'une signature valide ✅ |
| 7 | Détection de falsification (tamper detection) ❌ |

---

## Structure du projet

```
signtool/
├── README.md              ← Ce fichier
├── requirements.txt       ← Dépendances Python (click, cryptography, rich)
├── setup.py               ← Déclaration du paquet et du point d'entrée CLI
├── demo.bat               ← Script de démonstration Windows
├── demo.sh                ← Script de démonstration Linux/macOS
└── signtool/
    ├── __init__.py        ← Version et métadonnées du paquet
    ├── cli.py             ← Interface CLI (commandes Click + affichage Rich)
    ├── keygen.py          ← Génération de paires de clés RSA
    ├── signer.py          ← Logique de signature des fichiers
    └── verifier.py        ← Logique de vérification des signatures
```

---

## Sécurité

> ⚠️ Ces recommandations sont importantes pour garantir la sécurité de vos signatures.

- **Clé privée** : ne partagez **jamais** votre fichier `_private.pem`. Utilisez `--passphrase` pour le protéger par chiffrement AES.
- **Taille de clé** : RSA-2048 est le minimum recommandé. Préférez RSA-4096 pour des données sensibles ou une sécurité à long terme.
- **Algorithme** : SHA-256 + PKCS#1 v1.5 est largement supporté et approprié pour la plupart des usages. Pour les nouveaux systèmes, le padding PSS offre une sécurité accrue.
- **Horodatage** : le champ `timestamp` dans le `.sig` est indicatif et basé sur l'horloge de la machine signataire. Il ne constitue **pas** un horodatage certifié (TSA).
- **Distribution des clés** : distribuez votre clé publique via un canal sûr (site HTTPS officiel, empreinte vérifiée) pour éviter les attaques de substitution.

---

## Dépendances

| Paquet | Version minimale | Rôle |
|---|---|---|
| [`click`](https://click.palletsprojects.com/) | >= 8.1.0 | Framework CLI |
| [`cryptography`](https://cryptography.io/) | >= 41.0.0 | Génération de clés RSA, signature, vérification |
| [`rich`](https://rich.readthedocs.io/) | >= 13.0.0 | Affichage terminal coloré (tableaux, panneaux, spinners) |

---

## Licence

Ce projet est distribué sous licence **MIT**.  
Vous êtes libre de l'utiliser, le modifier et le redistribuer, à condition de conserver la notice de copyright d'origine.
