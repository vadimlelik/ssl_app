1- 📦 2. Dockerfile для Next.js

# Dockerfile

FROM node:18-alpine

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

COPY . .

RUN npm run build

EXPOSE 3000
CMD ["npm", "run", "start"]

⚙️ 3. Nginx конфиг
Создай папку nginx и файл default.conf внутри:

# nginx/default.conf

server {
listen 80;
server_name your-domain.com www.your-domain.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        proxy_pass http://next-app:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

}

🐳 4. Docker Compose

# Создай docker-compose.yml:

version: '3.8'

services:
next-app:
build: .
container_name: next-app
restart: always

nginx:
image: nginx:stable-alpine
container_name: nginx
ports: - "80:80" - "443:443"
volumes: - ./nginx:/etc/nginx/conf.d - ./certbot/www:/var/www/certbot - ./certbot/conf:/etc/letsencrypt
depends_on: - next-app
restart: always

certbot:
image: certbot/certbot
container_name: certbot
volumes: - ./certbot/conf:/etc/letsencrypt - ./certbot/www:/var/www/certbot

🛠️ # Шаг 7: Поднять проект (первый запуск — без HTTPS)
docker compose up -d --build

🔐 Шаг 8: Получить SSL-сертификаты
Выполни на сервере:
docker-compose run --rm certbot certonly \
 --webroot \
 --webroot-path=/var/www/certbot \
 -d your-domain.com -d www.your-domain.com \
 --email your-email@example.com \
 --agree-tos \
 --no-eff-email
✏️ Шаг 9: Обновить nginx/default.conf для HTTPS
server {
listen 80;
server_name your-domain.com www.your-domain.com;
return 301 https://$host$request_uri;
}

server {
listen 443 ssl;
server_name your-domain.com www.your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    location / {
        proxy_pass http://next-app:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

}
Перезапусти nginx:
docker compose restart nginx

🔁 Шаг 10: Настроить автоматическое обновление сертификатов

Добавь cron-задачу:
crontab -e
И вставь:
0 0 \* \* \* docker compose run --rm certbot renew --webroot --webroot-path=/var/www/certbot && docker compose exec nginx nginx -s reload

docker-compose run --rm certbot certonly \
 --webroot \
 --webroot-path=/var/www/certbot \
 -d cvirko-vadim.ru -d www.cvirko-vadim.ru \
 --email cvi-vadim@yandex.ru \
 --agree-tos \
 --no-eff-email
