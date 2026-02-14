# Outline Docker Compose

Self-hosted Outline wiki + documentation platform, ready for Docker Compose or Portainer deployment.

**What is Outline?**
- Modern team wiki with real-time collaboration
- REST + GraphQL API for integrations
- Markdown-based documents
- Multi-user with role-based permissions
- Fast full-text search
- Works great as your central knowledge base

---

## Quick Start (Docker Compose)

### 1. Clone the repo
```bash
git clone https://github.com/Gumbees/outline-docker.git
cd outline-docker
```

### 2. Generate SECRET_KEY
```bash
openssl rand -hex 32
```
Copy the output.

### 3. Create .env file
```bash
cp .env.example .env
```

Edit `.env` and replace:
- `SECRET_KEY=` with the value from step 2
- `OUTLINE_URL=` with your public URL (e.g., `http://your-ip:3000`)

### 4. Start the services
```bash
docker-compose up -d
```

### 5. Access Outline
Open browser to `http://localhost:3000`

First user created is automatically an admin. Set up your account and start documenting!

---

## Portainer Deployment

### Option 1: Using Portainer Stack (Recommended)

1. **Open Portainer** → Stacks → Add Stack
2. **Copy the content of `portainer-stack.yml`** into the editor
3. **Set Environment Variables:**
   - Click "Environment" → Add these:
     - `SECRET_KEY` = (generate with `openssl rand -hex 32`)
     - `OUTLINE_URL` = (your server IP/domain:3000)
4. **Click Deploy**

Portainer will spin up Outline + Redis with persistent volumes.

### Option 2: Upload docker-compose.yml

1. **In Portainer:** Stacks → Upload
2. **Select `docker-compose.yml` from this repo**
3. **Add environment variables** (as above)
4. **Click Deploy**

---

## Configuration

### .env Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SECRET_KEY` | Required | Session encryption key (generate: `openssl rand -hex 32`) |
| `OUTLINE_URL` | http://localhost:3000 | Public URL where users access Outline |
| `OUTLINE_PORT` | 3000 | Port exposed on host machine |
| `DATABASE_URL` | sqlite:///data/outline.db | SQLite path (recommended) |
| `REDIS_URL` | redis://redis:6379 | Redis connection |
| `FORCE_HTTPS` | false | Set to `true` if using HTTPS proxy |

### Optional Features

**Email Notifications (SMTP):**
```env
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=app-specific-password
SMTP_FROM_EMAIL=your-email@gmail.com
```

**Google OAuth:**
```env
GOOGLE_CLIENT_ID=your-id
GOOGLE_CLIENT_SECRET=your-secret
```

**GitHub OAuth:**
```env
GITHUB_CLIENT_ID=your-id
GITHUB_CLIENT_SECRET=your-secret
```

---

## Directory Structure

```
outline-docker/
├── docker-compose.yml          # Main services definition
├── portainer-stack.yml         # Portainer-ready stack
├── .env.example               # Environment variables template
├── README.md                  # This file
├── volumes/
│   ├── outline_data/          # SQLite DB + uploads (created on first run)
│   └── redis_data/            # Redis persistence (created on first run)
└── .gitignore                 # Excludes .env and node_modules
```

---

## First Run Checklist

- [ ] Generate `SECRET_KEY`: `openssl rand -hex 32`
- [ ] Create `.env` file from `.env.example`
- [ ] Set `OUTLINE_URL` to your public URL
- [ ] Run `docker-compose up -d`
- [ ] Wait 30 seconds for services to start
- [ ] Access Outline at configured URL
- [ ] Create your admin account (first user)
- [ ] Import docs or start creating collections

---

## Volumes & Persistence

**outline_data/**
- SQLite database file (`outline.db`)
- Uploaded files and attachments
- Persists even if containers are removed

**redis_data/**
- Redis session/cache data
- Automatically cleaned if expired
- Can be safely deleted (will rebuild)

Volumes are defined as Docker named volumes, so they persist across container restarts.

---

## Accessing the API

Outline has a powerful REST + GraphQL API. After login:

1. **Get API Token:**
   - Settings (gear icon) → API Tokens → Create Token
   - Copy the token

2. **Example API Call:**
```bash
curl -X GET http://localhost:3000/api/documents.list \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

See full API docs at: `http://localhost:3000/api/docs` (when running)

---

## Building Tools with Outline

Perfect for automation with N8N or custom scripts:

- **List all documents** → `POST /api/documents.list`
- **Create document** → `POST /api/documents.create`
- **Search documents** → `POST /api/documents.search`
- **Share document** → `POST /api/shares.create`
- **Update document** → `POST /api/documents.update`

Use the API to:
- Auto-generate docs from infrastructure code
- Sync data from other systems
- Build integrations with N8N workflows
- Trigger actions on document changes (webhooks)

---

## Troubleshooting

### Port already in use
```bash
# Change port in .env:
OUTLINE_PORT=3001
```

### Redis connection refused
```bash
# Ensure Redis is running:
docker-compose ps

# Restart services:
docker-compose down && docker-compose up -d
```

### Can't access from other machines
Update `OUTLINE_URL` in `.env` to use your server's IP/hostname:
```env
OUTLINE_URL=http://192.168.1.100:3000
```

### Database corrupted
Delete the volume and restart (you'll lose data):
```bash
docker-compose down
docker volume rm outline-docker_outline_data
docker-compose up -d
```

---

## Upgrading Outline

```bash
# Pull latest images
docker-compose pull

# Restart services
docker-compose up -d
```

Data persists in volumes, so upgrades are safe.

---

## Performance Notes

**SQLite** (this setup):
- ✅ Great for single-user or small teams (< 100 users)
- ✅ No extra database service needed
- ✅ Automatically backed up in volume
- ⚠️ Not ideal for 500+ concurrent users

For larger deployments, switch to PostgreSQL (change `DATABASE_URL`).

---

## Security

- Change `SECRET_KEY` for production
- Enable `FORCE_HTTPS=true` if using HTTPS reverse proxy
- Regularly backup `outline_data` volume
- Use OAuth (Google/GitHub) instead of passwords if possible
- Keep Outline image updated: `docker-compose pull`

---

## License & Credits

- **Outline** - https://github.com/outline/outline (AGPL-3.0)
- **This repo** - Docker Compose setup for easy deployment

---

## Support

For Outline issues: https://github.com/outline/outline/discussions
For deployment questions: Open an issue in this repo
