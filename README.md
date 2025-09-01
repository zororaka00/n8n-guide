# n8n Docker Setup with PostgreSQL and HTTPS via Nginx

This repository provides a step-by-step guide to set up n8n (a workflow automation tool) using Docker Compose with PostgreSQL as the database and Nginx as a reverse proxy for HTTPS.

## Prerequisites

- A Linux server with root access.
- Docker and Docker Compose installed. Follow the official guides:
  - [Docker Engine Installation](https://docs.docker.com/engine/install/)
  - [Docker Compose Installation](https://docs.docker.com/compose/install/linux/)
- A domain name with DNS configured (e.g., `subdomain.domain.com`).
- Nginx and Certbot for SSL setup.

## Installation

1. **Clone or Download the Repository**:
   ```bash
   git clone https://github.com/zororaka00/n8n-guide.git
   cd n8n-guide
   ```

2. **Create Environment File**:
   Create a `.env` file in the project directory with the following variables (adjust as needed):
   ```bash
   DOMAIN_NAME=domain.com
   SUBDOMAIN=subdomain
   GENERIC_TIMEZONE=Asia/Jakarta
   N8N_PROXY_HOPS=1

   POSTGRES_USER=username_db
   POSTGRES_PASSWORD=password_db
   POSTGRES_DB=n8n
   ```

3. **Set Up Docker Compose**:
   Use the provided `docker-compose.yaml` file (included in this repo) to run n8n and PostgreSQL:
   ```yaml
    version: '3.8'

    services:
      postgres:
        image: postgres:16
        restart: always
        environment:
          - POSTGRES_USER=${POSTGRES_USER}
          - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
          - POSTGRES_DB=${POSTGRES_DB}
        volumes:
          - postgres_data:/var/lib/postgresql/data

      n8n:
        image: docker.n8n.io/n8nio/n8n
        restart: always
        environment:
          - N8N_PROXY_HOPS=${N8N_PROXY_HOPS}
          - DB_TYPE=postgresdb
          - DB_POSTGRESDB_HOST=postgres
          - DB_POSTGRESDB_PORT=5432
          - DB_POSTGRESDB_DATABASE=${POSTGRES_DB}
          - DB_POSTGRESDB_USER=${POSTGRES_USER}
          - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}
          - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
          - N8N_RUNNERS_ENABLED=true
          - N8N_HOST=${SUBDOMAIN}.${DOMAIN_NAME}
          - N8N_PORT=5678
          - N8N_PROTOCOL=https
          - GENERIC_TIMEZONE=${GENERIC_TIMEZONE}
          - TZ=${GENERIC_TIMEZONE}
          - WEBHOOK_URL=https://${SUBDOMAIN}.${DOMAIN_NAME}
          - N8N_EDITOR_BASE_URL=https://${SUBDOMAIN}.${DOMAIN_NAME}
        ports:
          - 5678:5678
        depends_on:
          - postgres
        volumes:
          - n8n_data:/home/node/.n8n

    volumes:
      postgres_data:
      n8n_data:
    ```

4. **Run Docker Compose**:
   ```bash
   docker compose up -d
   ```

5. **Configure Nginx for HTTPS**:
   - Install Nginx and Certbot:
     ```bash
     sudo apt update
     sudo apt install nginx python3-certbot-nginx
     ```
   - Create an Nginx configuration file (e.g., `/etc/nginx/conf.d/n8n.conf`):
     ```nginx
     server {
         server_name subdomain.domain.com;

         location / {
             proxy_pass http://localhost:5678;
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection "upgrade";
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
             proxy_buffering off;
             proxy_cache off;
         }
     }
     ```
   - Obtain SSL certificate:
     ```bash
     sudo certbot --nginx -d subdomain.domain.com
     ```
   - Restart Nginx:
     ```bash
     sudo systemctl restart nginx
     ```

## Usage

- Access n8n at `https://subdomain.domain.com`.
- For production, ensure server security, backups, and monitoring.
- If you encounter "Connection lost" issues, verify WebSocket support in Nginx (add `proxy_set_header Upgrade $http_upgrade;` and `proxy_set_header Connection "upgrade";`).

## Updating

To update n8n:
```bash
docker compose down
docker compose pull
docker compose up -d
```

## Troubleshooting

- Refer to the [n8n Documentation](https://docs.n8n.io/) for more details.
- Check the [n8n Community Forum](https://community.n8n.io/) for common issues.

## Sources

- [n8n Docker Installation](https://docs.n8n.io/hosting/installation/docker/#using-with-postgresql)
- [n8n Blog: Setup via PM2](https://blog.n8n.io/how-to-set-up-n8n-via-pm2/#configure-nginx-and-ssl-certificate)

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Contributing

Feel free to submit
