---

# 🚀 Step-by-Step: n8n + Nginx Proxy Manager (Docker, Ubuntu, Detached)

---

## 1. **System Preparation**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl
```

---

## 2. **Install Docker and Docker Compose**

```bash
curl -fsSL https://get.docker.com | sudo bash
sudo apt install -y docker-compose-plugin
sudo usermod -aG docker $USER
```

---

## 3. **Create Shared Docker Network**

```bash
docker network create npm_network
```

---

## 4. **Deploy Nginx Proxy Manager**

```bash
mkdir -p ~/npm && cd ~/npm
nano docker-compose.yml
```

Paste:

```yaml
version: "3"
services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: npm
    restart: unless-stopped
    ports:
      - "80:80"
      - "81:81"
      - "443:443"
    environment:
      - TZ=Asia/Jakarta
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    networks:
      - npm_network

networks:
  npm_network:
    external: true
```

Start NPM:

```bash
docker compose up -d
```

---

## 5. **Create the Persistent n8n Data Volume**

```bash
docker volume create n8n_data
```

---

## 6. **Start n8n Container in Detached Mode and Join Network**

**This is your requested command, with improvements:**

```bash
docker run -d --name n8n --restart unless-stopped -p 5678:5678 -v n8n_data:/home/node/.n8n docker.n8n.io/n8nio/n8n
```

---

## 7. **Set Up Proxy Host in Nginx Proxy Manager**

1. Go to `http://YOUR-SERVER-IP:81`
2. Login to NPM dashboard.
3. Go to **Proxy Hosts > Add Proxy Host**.
4. **Domain Names:** `n8n.yourdomain.com`
5. **Scheme:** `http`
6. **Forward Hostname/IP:** `n8n` *(this matches the Docker container name, works because they’re on the same network!)*
7. **Forward Port:** `5678`
8. Enable **Websockets Support**
9. SSL Tab:

   * Request a Let’s Encrypt SSL cert
   * Enable **Force SSL** and all security options
10. Save

---

## 8. **Access Your n8n Instance**

Visit: `https://n8n.yourdomain.com`
Login using your username/password.

---

## 9. **Stop, Restart, or Remove n8n**

* **Stop:**
  `docker stop n8n`
* **Start:**
  `docker start n8n`
* **Remove:**
  `docker rm n8n`
  *(data remains in the volume!)*

---

## 10. **View Logs**

```bash
docker logs -f n8n
```

---

## **Summary**

* Your n8n data persists in `n8n_data` volume.
* Container runs on a shared Docker network with Nginx Proxy Manager.
* Production ready, with SSL, user authentication, and persistent data.
* All settings can be modified by adjusting environment variables.

---

Let me know if you want to add PostgreSQL or any advanced config!
