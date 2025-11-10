# n8n Setup with Rossmann Certificates

Complete guide for running n8n workflow automation with Rossmann self-signed certificates for accessing services like Azure OpenAI.

---

## 🎯 Key Features

✅ **Self-signed certificates** - Access Rossmann internal services  
✅ **PostgreSQL database** - Persistent workflow storage  
✅ **Docker volumes** - Data survives container restarts  
✅ **No telemetry errors** - Proper HTTPS certificate handling  
✅ **Production-ready** - Easy deployment with configuration changes

---

## 📋 Prerequisites

- Docker and Docker Compose installed
- WSL2 (if on Windows)
- Rossmann certificates in `.cert/` folder

## 🚀 Quick Start

### 1. Build the Custom n8n Image (one time)

```bash
# On Windows (PowerShell)
wsl docker build -f Dockerfile.n8n-rossmann -t n8n-rossmann .

# On Linux/WSL
docker build -f Dockerfile.n8n-rossmann -t n8n-rossmann .
```

### 2. Start n8n

```bash
# On Windows (PowerShell)
wsl docker-compose -f docker-compose.rossmann.yml up -d

# On Linux/WSL
docker-compose -f docker-compose.rossmann.yml up -d
```

**n8n is now accessible at:** <http://localhost:5678>


### 3. Stop n8n

```bash
# On Windows (PowerShell)
wsl docker-compose -f docker-compose.rossmann.yml down

# On Linux/WSL
docker-compose -f docker-compose.rossmann.yml down
```

---

## 📚 Common Commands

| Task | Windows (PowerShell) | Linux/WSL |
|------|---------------------|-----------|
| **Start n8n** | `wsl docker-compose -f docker-compose.rossmann.yml up -d` | `docker-compose -f docker-compose.rossmann.yml up -d` |
| **Stop n8n** | `wsl docker-compose -f docker-compose.rossmann.yml down` | `docker-compose -f docker-compose.rossmann.yml down` |
| **View logs** | `wsl docker-compose -f docker-compose.rossmann.yml logs -f n8n` | `docker-compose -f docker-compose.rossmann.yml logs -f n8n` |
| **Restart n8n** | `wsl docker-compose -f docker-compose.rossmann.yml restart` | `docker-compose -f docker-compose.rossmann.yml restart` |
| **Rebuild image** | `wsl docker build -f Dockerfile.n8n-rossmann -t n8n-rossmann .` | `docker build -f Dockerfile.n8n-rossmann -t n8n-rossmann .` |
| **Delete all data** | `wsl docker-compose -f docker-compose.rossmann.yml down -v` | `docker-compose -f docker-compose.rossmann.yml down -v` |

---

## 📁 Project Structure

```text
n8n/
├── .cert/                          # Rossmann SSL certificates
│   ├── root.crt                    # Root certificate authority
│   ├── rms-intern.crt              # ROSSMANN internal certificate
│   └── README.md                   # Certificate documentation
├── Dockerfile.n8n-rossmann         # Custom n8n image with certificates
├── docker-compose.rossmann.yml     # Docker Compose configuration
└── SETUP_ROSSMANN.md              # This file
```

*Everything else is the original n8n repository code.*

---

## 🔧 How It Works

### Custom Docker Image (`n8n-rossmann`)

Built from `Dockerfile.n8n-rossmann`:
- **Extends** the official `docker.n8n.io/n8nio/n8n:latest` image (not built from source)
- Copies certificates from `.cert/` directory
- Installs certificates into system certificate store
- Configures environment variables for Node.js, npm, and other tools

**Why this approach?**
- ✅ You don't need to rebuild n8n from source
- ✅ You get the official, tested n8n build
- ✅ You only add what's different (certificates)
- ✅ Much faster builds (seconds vs minutes)
- ✅ Easier to maintain and update

**When to rebuild:**
- Rossmann certificates are updated
- Want to use newer n8n version
- Certificate errors occur

**Where is the image stored?**

The custom `n8n-rossmann` image is stored **locally on your machine** in Docker's image cache (WSL2: `/var/lib/docker`).

**Storage options:**

1. **Local Docker cache** (current setup)
   - ✅ Quick access, no network required
   - ❌ Only available on your machine
   - ❌ Must rebuild on other machines
   - Good for: Development and testing

2. **Corporate Docker Registry** (recommended for production)
   - ✅ Shared across team
   - ✅ Version control and rollback
   - ✅ Pull from any machine
   - Good for: Production and team collaboration
   
   ```bash
   # Tag and push to registry
   wsl docker tag n8n-rossmann rossmann-registry.de/n8n-rossmann:1.0.0
   wsl docker push rossmann-registry.de/n8n-rossmann:1.0.0
   ```

3. **Docker Hub** (public)
   - ❌ Not recommended (contains internal certificates)

### Docker Compose Stack

The `docker-compose.rossmann.yml` orchestrates:

**PostgreSQL Database** (postgres:16-alpine)
- Stores workflows, credentials, execution history
- Data persisted in `postgres-data` volume

**n8n Application** (n8n-custom)
- Workflow automation platform
- Connected to PostgreSQL
- Rossmann certificates pre-installed
- Data persisted in `n8n-data` volume

**Networking**
- n8n and PostgreSQL communicate internally
- Only port 5678 exposed externally

### Persistent Storage

Docker volumes preserve data across restarts:
- `n8n-data` - Workflows, credentials, settings
- `postgres-data` - Database files

### Certificate Configuration

Environment variables in `docker-compose.rossmann.yml`:
- `NODE_EXTRA_CA_CERTS` - Node.js certificate trust
- `SSL_CERT_FILE` - General SSL operations
- `REQUESTS_CA_BUNDLE` - Python requests library
- `NODE_OPTIONS` - OpenSSL CA support
- `npm_config_cafile` - npm/pnpm package manager

---

## ✅ Verifying Your Configuration

### What Does Docker Compose Do?

**Docker Compose** orchestrates multiple containers as a single application. Instead of running complex `docker run` commands manually, it:

- ✅ Creates volumes automatically
- ✅ Creates a private network for services
- ✅ Starts containers in the right order
- ✅ Waits for dependencies (PostgreSQL healthcheck)
- ✅ Restarts containers if they crash
- ✅ Manages all environment variables
- ✅ Stops everything together with one command

### How to Verify Configuration is Correct

**1. Validate syntax and structure:**
```bash
wsl docker-compose -f docker-compose.rossmann.yml config
```
✅ If successful: Shows parsed configuration  
❌ If errors: Displays syntax problems

**2. Check image exists:**
```bash
wsl docker images | grep n8n-rossmann
```
✅ Should show: `n8n-rossmann   latest   ...`  
❌ If missing: Run the build command first

**3. Verify certificates in image:**
```bash
wsl docker run --rm n8n-rossmann ls -la /usr/local/share/ca-certificates/
```
✅ Should list: `root.crt` and `rms-intern.crt`

**4. Test startup (dry run):**
```bash
wsl docker-compose -f docker-compose.rossmann.yml up
```
✅ Watch logs for successful startup  
❌ Press Ctrl+C if errors appear

**5. Check running services:**
```bash
wsl docker-compose -f docker-compose.rossmann.yml ps
```
✅ Both services should show "Up" status

**6. Verify PostgreSQL connection:**
```bash
wsl docker-compose -f docker-compose.rossmann.yml exec postgres psql -U n8n -c "SELECT version();"
```
✅ Should show PostgreSQL version

**7. Test n8n web interface:**
- Open browser: http://localhost:5678
- ✅ Should load n8n setup page

---

## 🔍 Troubleshooting

### Port 5678 already in use

Edit `docker-compose.rossmann.yml`:

```yaml
ports:
  - "8080:5678"  # Change to port 8080
```

### Certificate errors accessing Azure/internal services

```bash
# Rebuild the image
wsl docker build -f Dockerfile.n8n-rossmann -t n8n-rossmann .

# Restart containers
wsl docker-compose -f docker-compose.rossmann.yml restart
```

### Verify certificates are installed

```bash
# Windows
wsl docker-compose -f docker-compose.rossmann.yml exec n8n sh -c "ls -la /usr/local/share/ca-certificates/"

# Linux/WSL
docker-compose -f docker-compose.rossmann.yml exec n8n sh -c "ls -la /usr/local/share/ca-certificates/"
```

Should show certificate files (root.crt, rms-intern.crt).

### View detailed logs

```bash
# Windows
wsl docker-compose -f docker-compose.rossmann.yml logs -f n8n

# Linux/WSL
docker-compose -f docker-compose.rossmann.yml logs -f n8n
```

### Access PostgreSQL directly

```bash
# Windows
wsl docker-compose -f docker-compose.rossmann.yml exec postgres psql -U n8n

# Linux/WSL
docker-compose -f docker-compose.rossmann.yml exec postgres psql -U n8n
```

### Complete reset

```bash
# Stop and delete all data
wsl docker-compose -f docker-compose.rossmann.yml down -v

# Rebuild and start fresh
wsl docker build -f Dockerfile.n8n-rossmann -t n8n-rossmann .
wsl docker-compose -f docker-compose.rossmann.yml up -d
```

---

## 🏭 Production Deployment

### Update Configuration

Edit `docker-compose.rossmann.yml` for production:

1. **Strong database password:**

   ```yaml
   POSTGRES_PASSWORD: <generate-strong-password>
   DB_POSTGRESDB_PASSWORD: <same-strong-password>
   ```

2. **Encryption key (important!):**

   ```yaml
   N8N_ENCRYPTION_KEY: <generate-random-32-char-key>
   ```

   Generate with: `openssl rand -base64 32`

3. **Production hostname:**

   ```yaml
   N8N_HOST: n8n.rossmann.de
   WEBHOOK_URL: https://n8n.rossmann.de/
   ```

4. **HTTPS support:**
   - Add nginx reverse proxy
   - Configure SSL/TLS certificates
   - Set up proper firewall rules

### Environment-Specific Configurations

**Test Environment:**
- Use test database credentials
- Enable debug logging: `N8N_LOG_LEVEL=debug`
- Test webhook URLs
- Limited retention of execution history

**Production Environment:**
- Strong passwords and encryption keys
- Disable debug logging
- Backup strategy for PostgreSQL
- Monitoring and alerting
- High availability setup (if needed)

---

## 📖 Next Steps

1. Access n8n at <http://localhost:5678>
2. Create your first workflow
3. Test connections to:
   - Azure OpenAI
   - Internal Rossmann APIs
   - External services
4. Read official documentation:
   - n8n docs: <https://docs.n8n.io>
   - Docker Compose: <https://docs.docker.com/compose/>

---

## 📝 Additional Notes

### Certificate Updates

When Rossmann certificates are renewed:

1. Replace files in `.cert/` directory
2. Rebuild Docker image
3. Restart containers

```bash
wsl docker build -f Dockerfile.n8n-rossmann -t n8n-rossmann .
wsl docker-compose -f docker-compose.rossmann.yml restart
```

### Backup Strategy

**Important data to backup:**
- PostgreSQL database (`postgres-data` volume)
- n8n settings and credentials (`n8n-data` volume)

**Backup command:**

```bash
# Backup volumes
wsl docker run --rm -v n8n_n8n-data:/data -v ${PWD}:/backup alpine tar czf /backup/n8n-data-backup.tar.gz /data
wsl docker run --rm -v n8n_postgres-data:/data -v ${PWD}:/backup alpine tar czf /backup/postgres-backup.tar.gz /data
```

### Performance Tuning

For high-volume workflows, consider:
- Increasing PostgreSQL connection pool
- Adjusting n8n execution concurrency
- Monitoring resource usage
- Scaling horizontally (multiple n8n instances)

---

## 🆘 Support

- **Internal:** Contact Rossmann IT team for certificate issues
- **n8n Community:** <https://community.n8n.io>
- **Documentation:** <https://docs.n8n.io>
