# 🚀 Dockerized LEMP Stack on CentOS Stream 9

This project sets up a complete **LEMP Stack** (Linux, Nginx, MySQL, PHP-FPM) using **Docker Compose** on a fresh **CentOS Stream 9** machine.

---

## 🛠️ Tech Stack & Services

| Service | Image / Tool | Port / Description |
| :--- | :--- | :--- |
| **OS** | CentOS Stream 9 | Host Environment |
| **Web Server** | Nginx (`latest`) | Port `8081` (Host) -> `80` (Container) |
| **Backend** | PHP-FPM (`8.2`) | Extensions: `mysqli`, `pdo`, `pdo_mysql` |
| **Database** | MySQL (`8.0`) | DB Name: `exampledb` |
| **Orchestration**| Docker Compose V2 | Multi-container Management |

---

## 📁 Project Structure

```text
lemp-project/
│
├── docker-compose.yml
├── nginx/
│   └── default.conf
├── php/
│   └── Dockerfile
└── www/
    └── index.php


📋 Prerequisites & Setup on CentOS Stream 9
Step 1: Update System & Remove Legacy Packages

sudo dnf update -y
sudo dnf remove -y docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine podman buildah


Step 2: Install Docker Engine & Docker Compose Plugin

# Install DNF plugins core
sudo dnf install -y dnf-plugins-core

# Add Docker Repository
sudo dnf config-manager --add-repo [https://download.docker.com/linux/centos/docker-ce.repo](https://download.docker.com/linux/centos/docker-ce.repo)

# Install Docker Packages
sudo dnf install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

Step 3: Start & Enable Docker Service

sudo systemctl start docker
sudo systemctl enable docker

Step 4: Configure User Permissions (Optional)
sudo usermod -aG docker $USER
newgrp docker

Step 5: Configure CentOS Firewall (Firewalld)
Allow incoming traffic on port 8081:

sudo firewall-cmd --permanent --add-port=8081/tcp
sudo firewall-cmd --reload


📄 Configuration Files
1. docker-compose.yml
(Note: :z flag is added to volume mounts for SELinux compatibility in CentOS 9)


services:
  nginx:
    image: nginx:latest
    ports:
      - "8081:80"
    volumes:
      - ./www:/var/www/html:z
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf:z
    depends_on:
      - php
    networks:
      - lempnet

  php:
    build: ./php
    volumes:
      - ./www:/var/www/html:z
    networks:
      - lempnet

  mysql:
    image: mysql:8.0
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: exampledb
      MYSQL_USER: user
      MYSQL_PASSWORD: userpassword
    volumes:
      - db_data:/var/lib/mysql
    networks:
      - lempnet

volumes:
  db_data:

networks:
  lempnet:



2. nginx/default.conf

server {
    listen 80;
    server_name localhost;

    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri$uri/ =404;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass php:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}



3. php/Dockerfile

FROM php:8.2-fpm
RUN docker-php-ext-install mysqli pdo pdo_mysql
WORKDIR /var/www/html



4. www/index.php

<?php
echo "<h1>LEMP Stack is Working on CentOS 9!</h1>";
$host = 'mysql';$db   = 'exampledb';
$user = 'user';$pass = 'userpassword';

$conn = new mysqli($host,$user, $pass,$db);
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "<p>Connected to MySQL successfully!</p>";
$conn->close();
?>

How to Run
Clone or navigate to the project directory:

cd lemp-project

Start the application in detached mode:

docker compose up -d
