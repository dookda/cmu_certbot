Obtaining and deploying an SSL certificate for Nginx running inside a Docker container involves several steps. Here's a guide to help you set up Certbot for an Nginx container using Docker:

### 1. Set Up Docker Compose

It's recommended to use Docker Compose to manage multi-container applications. Here's an example `docker-compose.yml` that starts both Nginx and Certbot:

```yaml
version: '3'

services:
  nginx:
    image: nginx:latest
    container_name: nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./data/nginx:/etc/nginx/conf.d
      - ./data/certbot/conf:/etc/letsencrypt
      - ./data/certbot/www:/var/www/certbot

  certbot:
    image: certbot/certbot
    volumes:
      - ./data/certbot/conf:/etc/letsencrypt
      - ./data/certbot/www:/var/www/certbot
```

### 2. Configure Nginx for Certbot

In your Nginx configuration (`nginx.conf`), you need to set up an HTTP server block to respond to the Certbot challenges. Create an `nginx.conf` file with the following content:

```nginx
http {
    server {
        listen 80;
        server_name yourdomain.com www.yourdomain.com;

        location ~ /.well-known/acme-challenge/ {
            allow all;
            root /var/www/certbot;
        }
    }
    # Additional configurations and server blocks
}
```

Replace `yourdomain.com` and `www.yourdomain.com` with your domain names.

### 3. Request SSL Certificates

With the services defined and Nginx configured to respond to the Certbot challenges, start the Nginx container:

```bash
docker-compose up -d nginx
```

Then, request the SSL certificates:

```bash
docker-compose run --rm certbot certonly --webroot -w /var/www/certbot -d yourdomain.com -d www.yourdomain.com
```

If successful, the certificates will be stored in the `./data/certbot/conf` directory on your host.

### 4. Update Nginx Configuration for SSL

Now, update your Nginx configuration (`nginx.conf`) to use the SSL certificates:

```nginx
http {
    server {
        listen 80;
        server_name yourdomain.com www.yourdomain.com;
        location / {
            return 301 https://$host$request_uri;
        }
    }

    server {
        listen 443 ssl;
        server_name yourdomain.com www.yourdomain.com;

        ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

        # Additional SSL configurations and server blocks
    }
    # ...
}
```

### 5. Restart Nginx

With the updated configurations, restart the Nginx container:

```bash
docker-compose down
docker-compose up -d
```

### 6. Set Up Certificate Renewal

Let's Encrypt certificates are valid for 90 days, so you need to set up an automatic renewal mechanism. This can be achieved by running the Certbot renewal command regularly using a cron job or other scheduling mechanisms.