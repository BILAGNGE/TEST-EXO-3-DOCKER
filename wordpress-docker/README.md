# WordPress Docker — Exercice 03

Déploiement de la dernière version de **WordPress** via `docker-compose`, en utilisant les images officielles **Nginx**, **MariaDB** et **PHP-FPM**.

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                     Docker Network                   │
│                      wp_network                      │
│                                                      │
│  ┌──────────┐    ┌──────────┐    ┌───────────────┐  │
│  │  nginx   │───▶│ php-fpm  │───▶│    mariadb    │  │
│  │ :80      │    │ :9000    │    │    :3306      │  │
│  └──────────┘    └──────────┘    └───────────────┘  │
│       │               │                  │           │
│       └───────────────┘                  │           │
│              wp_data (volume)    db_data (volume)    │
└─────────────────────────────────────────────────────┘
```

| Service | Image | Rôle |
|---------|-------|------|
| `nginx` | `nginx:latest` | Serveur web / reverse proxy, sert les fichiers statiques et délègue le PHP |
| `php` | `php:8.3-fpm` (custom) | Interprète PHP via FastCGI, contient les fichiers WordPress |
| `db` | `mariadb:latest` | Base de données relationnelle |

### Volumes partagés

| Volume | Contenu |
|--------|---------|
| `wp_data` | Fichiers WordPress (partagé entre `nginx` et `php`) |
| `db_data` | Données persistantes MariaDB |

---

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) ≥ 24
- [Docker Compose](https://docs.docker.com/compose/) ≥ 2.x (intégré à Docker Desktop)

---

## Installation et lancement

### 1. Cloner le dépôt

```bash
git clone https://github.com/<votre-user>/wordpress-docker.git
cd wordpress-docker
```

### 2. Configurer les variables d'environnement

```bash
cp .env.example .env
```

Éditer `.env` et **changer les mots de passe** :

```env
MYSQL_ROOT_PASSWORD=un_mot_de_passe_fort
MYSQL_DATABASE=wordpress
MYSQL_USER=wp_user
MYSQL_PASSWORD=un_autre_mot_de_passe_fort
```

> ⚠️ Le fichier `.env` est listé dans `.gitignore` — il ne sera jamais commité.

### 3. Construire et démarrer les conteneurs

```bash
docker compose up -d --build
```

La première exécution peut prendre quelques minutes (téléchargement des images + WordPress).

### 4. Finaliser l'installation de WordPress

Ouvrir un navigateur sur **[http://localhost](http://localhost)** et suivre l'assistant d'installation WordPress :

- **Hôte de la base de données** : `db` *(nom du service Docker, pas localhost)*
- **Nom de la base** : valeur de `MYSQL_DATABASE`
- **Utilisateur** : valeur de `MYSQL_USER`
- **Mot de passe** : valeur de `MYSQL_PASSWORD`

---

## Commandes utiles

```bash
# Voir l'état des conteneurs
docker compose ps

# Suivre les logs en temps réel
docker compose logs -f

# Arrêter sans supprimer les données
docker compose stop

# Arrêter ET supprimer les conteneurs (les volumes sont conservés)
docker compose down

# Tout supprimer, y compris les volumes (⚠️ perte de données)
docker compose down -v

# Reconstruire uniquement l'image PHP
docker compose build php
```

---

## Structure du projet

```
wordpress-docker/
├── docker-compose.yaml      # Orchestration des services
├── .env.example             # Template des variables d'environnement
├── .env                     # Variables réelles (non versionné)
├── .gitignore
├── nginx/
│   └── default.conf         # Configuration du vhost Nginx
├── php/
│   └── Dockerfile           # Image PHP-FPM + extensions + WordPress
└── README.md
```

---

## Détails techniques

### Nginx (`nginx/default.conf`)
- Écoute sur le port **80**
- `try_files` pour les permaliens WordPress
- Délègue les `.php` à `php:9000` via **FastCGI**
- Cache navigateur pour les assets statiques
- Taille max d'upload : **64 Mo**

### PHP (`php/Dockerfile`)
- Base `php:8.3-fpm`
- Extensions installées : `gd`, `mysqli`, `pdo_mysql`, `zip`, `intl`, `opcache`, `mbstring`, `exif`
- WordPress téléchargé depuis `wordpress.org/latest.tar.gz` à la construction de l'image
- `upload_max_filesize` et `post_max_size` alignés avec Nginx (64 Mo)

### MariaDB
- Image officielle `mariadb:latest`
- Credentials injectés via variables d'environnement (fichier `.env`)
- Données persistées dans le volume `db_data`
