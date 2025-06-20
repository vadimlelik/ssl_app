docker-compose run --rm certbot certonly \
 --webroot \
 --webroot-path=/var/www/certbot \
 -d cvirko-vadim.ru -d www.cvirko-vadim.ru \
 --email cvi-vadim@yandex.ru \
 --agree-tos \
 --no-eff-email
