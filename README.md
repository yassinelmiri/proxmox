# 🚀 Documentation d'installation 
## Déploiement sur un serveur Proxmox Project Coursa

---

# 📌 1. Prérequis

Serveur Proxmox : [https://ns3107703.ip-54-36-179.eu:8006](https://ns3107703.ip-54-36-179.eu:8006)

* Serveur Proxmox VE installé
* Un conteneur **LXC Debian/Ubuntu** ou une VM


* Docker & Docker Compose installés sur la VM/LXC
* Vos projets frontend et backend déjà poussés sur GitHub

https://github.com/melotrex/coursa-backend
https://github.com/melotrex/coursa_frontend_senegal


* Connecter sur ubuntu est update && apgrade 


<img width="532" height="307" alt="image" src="https://github.com/user-attachments/assets/7e8faa90-f71a-4769-a166-38be1ddeafc5" />

install tout les tool de project comme node docker ect 

<img width="527" height="342" alt="image" src="https://github.com/user-attachments/assets/31d8721f-2d3b-4e8e-9a58-cedfd4627a50" />

verifer les installation tool --version comme sur image 

<img width="223" height="96" alt="image" src="https://github.com/user-attachments/assets/74581638-3f87-4c0f-b130-5ec241602a90" />



---

# 📌 2. Installer Docker et Docker Compose

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ca-certificates curl gnupg -y

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/keyrings/docker.asc > /dev/null

echo \"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable\" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
```

Vérifier :

```bash
docker --version
docker compose version
```

---

# 📌 3. Cloner vos projets GitHub sur le serveur Proxmox

### 🔹 Backend NestJS

```bash
git clone https://github.com/melotrex/coursa-backend.git
cd backend-nest
```

### 🔹 Frontend Next.js

```bash
git clone https://github.com/melotrex/coursa_frontend_senegal.git
cd frontend-next
```

---

# 📌 4. Exemple de structure Docker

### 🔹 Dockerfile (Backend NestJS)

```Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install --production
COPY . .
CMD [\"npm\", \"run\", \"start:prod\"]
```

### 🔹 Dockerfile (Frontend Next.js)

```Dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
CMD [\"npm\", \"start\"]
```

---

# 📌 5. docker-compose.yml (Backend + Frontend + PostgreSQL)

```yaml
version: '3.8'
services:
  backend:
    build: ./backend-nest
    container_name: nest_api
    ports:
      - "3001:3001"
    depends_on:
      - db
    environment:
      DATABASE_URL: postgres://postgres:admin@db:5432/app
    restart: always

  frontend:
    build: ./frontend-next
    container_name: next_front
    ports:
      - "3000:3000"
    restart: always

  db:
    image: postgres:15
    container_name: pg_db
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: admin
      POSTGRES_DB: app
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

---

# 📌 6. Lancer les services Docker

```bash
docker compose up -d --build
```

Vérifier les conteneurs :

```bash
docker ps
```

---

# 📌 7. Configuration DNS / Reverse Proxy (Optionnel mais recommandé)

Installer **NGINX Proxy Manager** :

```bash
docker compose up -d
```

Créer un hôte :

* frontend : [https://app.votredomaine.com](https://app.votredomaine.com) → port 3000
* backend : [https://api.votredomaine.com](https://api.votredomaine.com) → port 3001

---

# 📌 8. Déployer dans Proxmox avec un LXC/VM

1. Créer une VM Debian/Ubuntu
2. Donner 2 Go RAM minimum
3. Installer Docker
4. Cloner vos projets GitHub
5. Lancer `docker compose up -d`

---

# 📌 9. Mise à jour après push GitHub

### 🔹 Mise à jour du backend

```bash
cd backend-nest
git pull
docker compose up -d --build backend
```

### 🔹 Mise à jour du frontend

```bash
cd frontend-next
git pull
docker compose up -d --build frontend
```

---

# 📌 9. Installation complète depuis Proxmox (Étape par Étape)

## 🟦 Étape 1 — Installer une VM Ubuntu dans Proxmox

1. Se connecter à votre Proxmox : `https://ns3107703.ip-54-36-179.eu:8006`
2. Télécharger une ISO **Ubuntu Server 22.04 LTS** (ou 24.04)
3. Aller dans : `Datacenter → Storage (local) → ISO Images → Upload`
4. Créer une nouvelle VM :

   * OS : Ubuntu ISO
   * CPU : 2 vCPU
   * RAM : 2 à 4 Go
   * Disque : 20 Go recommandé
   * Réseau : Bridge `vmbr0`

Démarrer la VM puis installer Ubuntu normalement.

---

## 🟦 Étape 2 — Installer Docker + Docker Compose sur Ubuntu

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install ca-certificates curl gnupg git -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/keyrings/docker.asc >/dev/null
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list >/dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
```

Vérifier :

```bash
docker --version
docker compose version
```

---

## 🟦 Étape 3 — Installer Node.js & NPM (si besoin hors Docker)

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
node -v
npm -v
```

---

## 🟦 Étape 4 — Créer un LXC pour exécuter les projets (optionnel)

👉 Si tu préfères tout mettre dans une VM, ignore cette étape.

### 1. Télécharger template Ubuntu pour LXC

Proxmox → `local → CT Templates → Templates → Ubuntu 22.04`

### 2. Créer un conteneur LXC

* OS : Ubuntu Template
* CPU : 2
* RAM : 2–4 Go
* Storage : 8–16 Go
* Network : Bridge `vmbr0`

### 3. Entrer dans le LXC

```bash
pct enter 100
```

(100 = ID du conteneur)

### 4. Installer Docker dans le LXC

Même commandes que plus haut.

---

## 🟦 Étape 5 — Cloner vos projets GitHub

### Backend NestJS

```bash
git clone https://github.com/melotrex/coursa-backend
cd coursa-backend
```

### Frontend Next.js

```bash
git clone https://github.com/melotrex/coursa_frontend_senegal
cd coursa_frontend_senegal
```

---

## 🟦 Étape 6 — Lancer les projets avec Docker

Dans le répertoire qui contient `docker-compose.yml` :

```bash
docker compose up -d --build
```

Si backend et frontend sont séparés :

### Backend

```bash
cd coursa-backend
docker compose up -d --build
```

### Frontend

```bash
cd coursa_frontend_senegal
docker compose up -d --build
```

---

## 🟦 Étape 7 — Vérifier les services

```bash
docker ps
```

Tu dois voir :

* NestJS → port 3001
* NextJS → port 3000
* PostgreSQL / MySQL (selon ton projet)

---

## 📌 10. Conclusion

Vous avez maintenant un environnement complet **NextJS + NestJS + PostgreSQL** entièrement déployé avec Docker sur un serveur Proxmox.

Si tu veux, je peux aussi :

* générer une version plus professionnelle
* ajouter des schémas d'architecture
* ajouter une section CI/CD GitHub Actions
* ajouter un script automatique d'installation
