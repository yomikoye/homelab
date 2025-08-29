# Homelab

A modular Docker-based homelab setup with organized service stacks for infrastructure, media, AI, and container management.

## Stack Overview

### 📁 **infra** - Core Infrastructure
- **Pi-hole**: Network-wide ad blocking and DNS
- **Traefik**: Reverse proxy with automatic SSL certificates

### 🎬 **media** - Media Services  
- **Jellyfin**: Media streaming server for movies, TV shows, and music

### ☁️ **nextcloud** - Personal Cloud
- **Nextcloud**: Personal cloud storage and collaboration platform

### 🤖 **llm** - AI Services
- **Open WebUI**: Interface for local AI/LLM models

### 🐳 **portainer** - Container Management
- **Portainer**: Docker container management interface

### 📊 **monitoring** - Health & Monitoring
- **Beszel**: Infrastructure health monitoring and rollout service

## Service URLs

| Service | Stack | URL | Purpose |
|---------|-------|-----|---------|
| Pi-hole | infra | `https://pihole.home` | DNS & Ad blocking |
| Traefik | infra | `https://traefik.home` | Reverse proxy dashboard |
| Jellyfin | media | `https://jellyfin.home` | Media streaming |
| Nextcloud | nextcloud | `https://nextcloud.home` | Personal cloud |
| Open WebUI | llm | `https://chat.home` | AI interface |
| Portainer | portainer | `https://portainer.home` | Container management |
| Beszel | monitoring | `https://health.koye.casa` | Health monitoring |

## Prerequisites

- Docker and Docker Compose installed
- Shared storage mounted at `/mnt/shared`
- Domain resolution for `*.home` domains (Pi-hole or hosts file)

## Quick Start

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd homelab
   ```

2. **Start infrastructure first**
   ```bash
   cd infra
   docker-compose up -d
   cd ..
   ```

3. **Deploy other stacks**
   ```bash
   # Media stack
   cd media
   docker-compose up -d
   cd ..
   
   # Personal cloud
   cd nextcloud
   docker-compose up -d
   cd ..
   
   # AI services
   cd llm  
   docker-compose up -d
   cd ..
   
   # Container management
   cd portainer
   docker-compose up -d
   
   # Monitoring services
   cd monitoring
   docker-compose up -d
   ```

## Nextcloud Stack Setup

1. **Create required directories**
   ```bash
   sudo mkdir -p /mnt/shared/portainer/nextcloud/{db,html,data}
   sudo mkdir -p /mnt/shared/nextcloud-files
   ```

2. **Set proper permissions**
   ```bash
   # Nextcloud directories need www-data ownership
   sudo chown -R www-data:www-data /mnt/shared/portainer/nextcloud/html
   sudo chown -R www-data:www-data /mnt/shared/portainer/nextcloud/data  
   sudo chown -R www-data:www-data /mnt/shared/nextcloud-files
   sudo chmod -R 755 /mnt/shared/portainer/nextcloud/{html,data}
   sudo chmod -R 755 /mnt/shared/nextcloud-files
   ```

3. **Configure environment variables** (⚠️ **Important**)
   
   Set these environment variables in Portainer or create a `.env` file:
   ```env
   NEXTCLOUD_DB_ROOT_PASSWORD=your_secure_root_password
   NEXTCLOUD_DB_PASSWORD=your_secure_db_password  
   NEXTCLOUD_ADMIN_PASSWORD=your_secure_admin_password
   ```

4. **Complete setup via web interface**
   
   After starting the containers, access `https://nextcloud.home` to complete installation:
   - Create admin account (use your `NEXTCLOUD_ADMIN_PASSWORD` value)
   - **Important**: Choose **MySQL/MariaDB** as database (not SQLite)
   - Database settings:
     - Database user: `nextcloud`
     - Database password: Your `NEXTCLOUD_DB_PASSWORD` value
     - Database name: `nextcloud`
     - Database host: `nextcloud-db:3306`

## Configuration

### Network Architecture
- **homelab-network**: External Docker network created by infrastructure stack
- All services connect to this shared network for inter-service communication
- Traefik handles SSL termination and routing for all `*.home` domains

### Stack Dependencies
1. **infra** must be deployed first (creates network and handles routing)
2. **media**, **nextcloud**, **llm**, and **portainer** can be deployed in any order after infra

### Environment Variables
All services are configured for **Europe/Warsaw** timezone. Update the `TZ` environment variable in each stack if needed.

### Volume Mappings
- **Media files**: `/mnt/shared/media` → Jellyfin media library (read-only)
- **Cloud storage**: `/mnt/shared/nextcloud-files` → Nextcloud user files
- **Config persistence**: `/mnt/shared/portainer/` → Service configurations

### Jellyfin (Media Stack)
- Hardware acceleration enabled (`/dev/dri` device mapping)
- Auto-discovery and DLNA support
- Published server URL configured for external access

### Nextcloud (Nextcloud Stack)
- MariaDB 10.11 database backend
- HTTPS protocol override for proper reverse proxy handling
- Trusted domain configured for `nextcloud.home`
- Admin user: `admin` (change the password!)

### Pi-hole (Infrastructure Stack)
- Network-wide DNS and ad blocking
- Web interface for managing blocklists and queries
- Custom DNS entries for local services

### Traefik (Infrastructure Stack)
- Automatic SSL certificate management
- Dynamic service discovery via Docker labels
- Dashboard for monitoring routes and services

## Stack Management

### Individual stack operations
```bash
# Start a specific stack
cd [stack-name]
docker-compose up -d

# View stack logs
cd [stack-name] 
docker-compose logs -f

# Update stack services
cd [stack-name]
docker-compose pull
docker-compose up -d

# Stop a stack
cd [stack-name]
docker-compose down
```

### Full homelab operations
```bash
# Start all stacks (in correct order)
./deploy.sh

# Check status across all stacks
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

### Backing up data
Important directories to backup by stack:
- **infra**: `/mnt/shared/portainer/traefik/`, `/mnt/shared/portainer/pihole/`
- **media**: `/mnt/shared/portainer/jellyfin/`, `/mnt/shared/media/`
- **nextcloud**: `/mnt/shared/portainer/nextcloud/`, `/mnt/shared/nextcloud-files/`
- **llm**: `/mnt/shared/portainer/openwebui/`
- **portainer**: `/mnt/shared/portainer/portainer/`
- **monitoring**: `/mnt/shared/portainer/beszel/`

## Troubleshooting

### Services not accessible
1. Verify infra stack is running: `cd infra && docker-compose ps`
2. Check `homelab-network` exists: `docker network ls`
3. Confirm DNS resolution for `*.home` domains (Pi-hole admin interface)
4. Check Traefik dashboard for service registration

### Stack dependencies
- If services can't communicate, ensure infra stack created `homelab-network`
- Restart dependent stacks if network was recreated

### Nextcloud specific issues

### Permission issues
```bash
# Check file ownership
ls -la /mnt/shared/portainer/nextcloud/

# Fix Nextcloud permissions
sudo chown -R 33:33 /mnt/shared/portainer/nextcloud/{html,data}
sudo chown -R 33:33 /mnt/shared/nextcloud-files/
```

### Database connection issues
1. Check MariaDB container logs: `docker-compose logs nextcloud-db`
2. Verify database credentials in environment variables
3. Ensure database container started before Nextcloud

## Security Notes

- **Change all default passwords** before deployment
- Consider using Docker secrets for sensitive data
- Implement regular backups of configuration and data
- Keep services updated with latest security patches

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test the configuration
5. Submit a pull request

## License

This homelab configuration is provided as-is for educational and personal use.