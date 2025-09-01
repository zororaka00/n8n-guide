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
   git clone https://github.com/yourusername/your-repo-name.git
   cd your-repo-name
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
   Use the provided `docker-compose.yaml` file (included in this repo) to run n8n and PostgreSQL.

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
