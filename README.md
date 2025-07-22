# fdl-poc

```
sudo docker compose down
```

```
# HTTP Server Block: Handles ACME challenge and redirects to HTTPS
server {
    listen 80;
    server_name link.zylobin.com;
    # Route for ACME challenges (Let's Encrypt validation)
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }
    # Initially serve the application on HTTP until we get certificates
    location / {
        proxy_pass http://client:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /api/ {
        proxy_pass http://server:5001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

```
nginx: # Nginx reverse proxy
    image: nginx:alpine
    container_name: dynamiclinks-nginx
    ports:
      - "80:80" # Expose port 80 to the host
      - "443:443"
    volumes:
      - ./nginx/init-nginx.conf:/etc/nginx/conf.d/default.conf # Starting with HTTP-only config
      - ./nginx/default.conf:/etc/nginx/conf.d/https.conf.disabled # Keep the HTTPS config but don't use yet
      - certbot_etc:/etc/letsencrypt:rw
      - certbot_www:/var/www/certbot:rw
    depends_on:
      - client
      - server
    command: "/bin/sh -c 'while :; do sleep 6h & wait $${!}; nginx -s reload; done & nginx -g \"daemon off;\"'"
```

```
sudo docker compose up --build -d
```

```
sudo docker compose exec certbot certbot certonly --webroot -w /var/www/certbot -d link.zylobin.com --email dimuthu.billa@gmail.com --agree-tos --no-eff-email --force-renewal
```

change back rebuild again
# fdl-poc
