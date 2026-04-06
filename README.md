# 🚀 NGINX Deep Guide (Line-by-Line + Best Practices)

## 📖 Overview
NGINX is a high-performance HTTP server and reverse proxy designed to handle large volumes of concurrent connections efficiently. It is commonly used as a gateway in front of backend services.

---

## 🧩 Minimal Working Configuration

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

## 🔍 Line-by-Line Explanation

### events {}
- Core event system (connection handling)
- Usually left empty unless tuning performance
- Handles worker connections internally

---

### http {}
- Main HTTP configuration block
- All web server logic lives here
- Supports:
  - servers
  - logging
  - compression
  - caching

---

### server {}
- Defines a virtual server (like a domain or app)
- You can have multiple server blocks for different domains

---

### listen 80;
- NGINX listens on port 80 (HTTP)
- Standard web traffic port

---

### location /
- Matches all incoming requests
- `/` = root path (everything)

---

### proxy_pass http://api:4000;
- Forwards requests to backend service
- `api` = hostname (can be container or server)
- `4000` = backend port

👉 Key behavior:
- NGINX becomes middle layer (reverse proxy)

---

### proxy_set_header Host $host;
- Pass original host (domain) to backend
- Required for:
  - virtual hosts
  - authentication systems

---

### proxy_set_header X-Real-IP $remote_addr;
- Sends real client IP to backend
- Without this → backend sees only NGINX IP

---

### proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
- Keeps full chain of client + proxies
- Example:
  client → proxy1 → proxy2 → nginx → backend

👉 Backend receives:
X-Forwarded-For: client_ip, proxy1_ip, proxy2_ip

---

### proxy_set_header X-Forwarded-Proto $scheme;
- Tells backend if request is HTTP or HTTPS
- Important for:
  - redirects
  - secure cookies

---

## ⚡ Best Practices

### 🔒 1. Always preserve real client IP
Use:
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

---

### 🔐 2. Always pass protocol
proxy_set_header X-Forwarded-Proto $scheme;

---

### 🚫 3. Never expose backend directly
❌ Bad:
User → API (port 4000)

✅ Good:
User → NGINX → API

---

### 🚦 4. Add rate limiting

limit_req_zone $binary_remote_addr zone=api_limit:10m rate=100r/m;

server {
    location / {
        limit_req zone=api_limit burst=20 nodelay;
    }
}

---

### ⚙️ 5. Use proper timeouts

proxy_connect_timeout 5s;
proxy_send_timeout 10s;
proxy_read_timeout 10s;

---

### 📦 6. Enable gzip (performance)

gzip on;
gzip_types text/plain application/json text/css application/javascript;

---

## 🧪 Example Configs

---

### ✅ Basic Reverse Proxy

server {
    listen 80;

    location / {
        proxy_pass http://localhost:4000;
    }
}

---

### ✅ API + Static Files

server {
    listen 80;

    location /api/ {
        proxy_pass http://localhost:4000;
    }

    location / {
        root /var/www/html;
        index index.html;
    }
}

---

### ✅ Load Balancing

http {
    upstream backend {
        server localhost:4000;
        server localhost:4001;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}

---

### ✅ HTTPS Redirect

server {
    listen 80;
    return 301 https://$host$request_uri;
}

---

### ✅ HTTPS Server

server {
    listen 443 ssl;

    ssl_certificate /etc/ssl/cert.pem;
    ssl_certificate_key /etc/ssl/key.pem;

    location / {
        proxy_pass http://localhost:4000;
    }
}

---

## 🐞 Common Mistakes

❌ Missing headers → wrong client IP  
❌ Wrong proxy_pass path → broken routing  
❌ No HTTPS → insecure app  
❌ No rate limiting → vulnerable to abuse  

---

## 📌 Summary

NGINX is not just a web server — it is:
- A reverse proxy
- A traffic controller
- A security layer
- A performance optimizer

Use it properly and your backend becomes:
- More secure
- More scalable
- More production-ready