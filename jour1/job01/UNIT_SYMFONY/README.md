# Configuration de l'Environnement Symfony

Ce document détaille la configuration initiale de l'environnement de développement pour un projet Symfony.

## Prérequis

### 1. Docker et Docker Compose

Docker et Docker Compose sont nécessaires pour créer et gérer notre environnement de développement conteneurisé.

Vérification des versions installées :

![Docker Versions](./images/1.docker_version.png)

### 2. Composer

Composer est le gestionnaire de dépendances PHP utilisé par Symfony.

Vérification de la version installée :

![Composer Version](./images/2.composer_version.png)

## Structure du Projet

```
jour1/
└── job01/
    └── UNIT_SYMFONY/       # Dossier principal du projet
        ├── images/         # Dossier contenant les captures d'écran
        └── README.md       # Documentation du projet
```

## Configuration Docker

### 1. docker-compose.yml

Le fichier `docker-compose.yml` définit l'infrastructure complète de notre application avec 5 services :

#### Service: app
```yaml
app:
  image: php:8.2-fpm          # Utilise l'image PHP 8.2 avec FPM
  build:                      # Configuration de build
    context: .                # Contexte de build (dossier courant)
    dockerfile: Dockerfile    # Utilise notre Dockerfile personnalisé
  container_name: symfony_app # Nom du conteneur
  working_dir: /var/www/html  # Répertoire de travail dans le conteneur
  volumes:
    - ./app:/var/www/html    # Monte le code source dans le conteneur
  networks:
    - symfony_network        # Connecte au réseau symfony_network
```

#### Service: webserver
```yaml
webserver:
  image: nginx:stable        # Utilise l'image stable de Nginx
  container_name: symfony_webserver
  ports:
    - "8080:80"             # Expose le port 8080 en local, mappé au port 80 du conteneur
  volumes:
    - ./app:/var/www/html   # Monte le code source
    - ./nginx:/etc/nginx/conf.d  # Monte la configuration Nginx
  depends_on:
    - app                   # Démarre après le service app
```

#### Service: database
```yaml
database:
  image: mysql:8.0          # Utilise MySQL 8.0
  environment:              # Variables d'environnement pour MySQL
    MYSQL_ROOT_PASSWORD: root
    MYSQL_DATABASE: symfony
    MYSQL_USER: symfony
    MYSQL_PASSWORD: symfony
  ports:
    - "3306:3306"          # Expose MySQL sur le port standard
  volumes:
    - db_data:/var/lib/mysql  # Stockage persistant des données
```

#### Services: adminer et phpmyadmin
```yaml
adminer:
  image: adminer
  ports:
    - "8081:8080"          # Interface Adminer sur http://localhost:8081

phpmyadmin:
  image: phpmyadmin/phpmyadmin
  ports:
    - "8082:80"            # Interface phpMyAdmin sur http://localhost:8082
```

### 2. Dockerfile

```dockerfile
FROM php:8.2-fpm

# Installer Composer
RUN apt-get update && apt-get install -y curl unzip git
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer
```
Ce Dockerfile :
- Utilise PHP 8.2 FPM comme image de base
- Installe les dépendances nécessaires (curl, unzip, git)
- Installe Composer globalement

### 3. nginx/default.conf

```nginx
server {
    listen 80;                # Écoute sur le port 80
    server_name localhost;    # Nom du serveur
    root /var/www/html/public;  # Racine du projet Symfony

    location / {
        try_files $uri /index.php$is_args$args;  # Redirige vers index.php
    }

    location ~ \.php$ {
        fastcgi_pass app:9000;  # Passe les requêtes PHP au service app
        # Configuration FastCGI standard
    }
}
```
Cette configuration Nginx :
- Définit le point d'entrée pour l'application Symfony
- Configure le traitement des fichiers PHP via PHP-FPM
- Gère les routes Symfony correctement

## Structure du Projet Mise à Jour

```
UNIT_SYMFONY/
├── app/                    # Code source Symfony (créé ultérieurement)
├── nginx/                  # Configuration Nginx
│   └── default.conf
├── docker-compose.yml      # Configuration des services
├── Dockerfile             # Configuration de l'image PHP
├── images/                # Captures d'écran
└── README.md              # Documentation
```

## Étapes Suivantes

Les prochaines étapes consisteront à :
1. Créer un nouveau projet Symfony
2. Configurer Docker pour notre environnement de développement
3. Mettre en place la base de données

---
*Documentation créée le 18 février 2025*
