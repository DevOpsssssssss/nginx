# 🚀 NGINX Setup Guide (Production Ready)

## 📖 Overview
NGINX is a high-performance web server that also works as a reverse proxy, load balancer, and HTTP cache. It acts as the entry point of your system, handling all incoming requests and forwarding them to your backend.

---

## 🏗️ Architecture

Client → NGINX → Backend API (Node.js) → Database

- Client: Browser or mobile app  
- NGINX: Handles routing, security, and performance  
- Backend API: Your app (e.g., Express on port 4000)  
- Database: MongoDB, PostgreSQL, etc.  

---

## ⚙️ Core Features

### 🌐 Web Server
NGINX can serve static files like HTML, CSS, JavaScript, and images directly to users.

### 🔁 Reverse Proxy
NGINX forwards requests from users to your backend service:
Client → NGINX → API

### ⚖️ Load Balancer
NGINX can distribute traffic across multiple backend servers:
Client → NGINX → API1 / API2 / API3

### 🔒 HTTPS / SSL
NGINX handles HTTPS encryption:
Client (HTTPS) → NGINX → Backend (HTTP)

### 🚦 Rate Limiting
NGINX can limit requests per user to prevent abuse and protect your API.

---

## 🧩 Basic NGINX Configuration

---

events {}

http {
    server {
        listen 80;

        location / {
            proxy_pass http://api:4000;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}

---

## 🐳 Docker Setup

version: "3.9"

services:
  api:
    build: .
    container_name: api
    restart: unless-stopped
    expose:
      - "4000"

  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api

---

## ▶️ How to Run

docker compose up -d --build

Open in browser:
http://localhost

---

## 🔒 Enable HTTPS (Production)

sudo apt install certbot python3-certbot-nginx  
sudo certbot --nginx -d yourdomain.com  

---

## 🧠 Best Practices

- Do NOT expose backend ports (e.g., 4000)  
- Always use NGINX in front of your API  
- Enable HTTPS in production  
- Apply rate limiting  
- Enable logging for debugging  
- Use environment variables for configuration  

---

## 🐞 Troubleshooting

Check NGINX logs:
docker logs nginx  

Check API logs:
docker logs api  

Test NGINX config:
nginx -t  

---

## 📌 Summary

NGINX improves performance, security, and scalability by acting as a gateway between users and your backend system. It is an essential component for production-ready applications.

---

## 📚 Next Steps

- Add domain + HTTPS  
- Configure caching  
- Add load balancing  
- Integrate monitoring (Prometheus + Grafana)  