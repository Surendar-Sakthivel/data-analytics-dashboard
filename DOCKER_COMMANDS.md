# Docker Commands - Quick Reference

## Where to Run Commands

**Always run from project directory:**
```bash
cd ~/Desktop/data-analytics-dashboard
```

---

## Daily Commands

### Start Databases
```bash
docker-compose up -d
```

### Stop Databases
```bash
docker-compose stop
```

### Check Status
```bash
docker-compose ps
```

### View Logs
```bash
docker-compose logs
docker-compose logs -f          # Follow logs (live)
docker-compose logs postgres    # Specific service
```

### Restart Databases
```bash
docker-compose restart
```

---

## Connect to Databases

### PostgreSQL
```bash
docker-compose exec postgres psql -U analytics_user -d analytics_db
```

### MongoDB
```bash
docker-compose exec mongodb mongosh -u analytics_user -p analytics_password
```

---

## Delete Everything (Start Fresh)

### Stop and Remove Containers (Keeps Data)
```bash
docker-compose down
```

### Stop and Remove Containers + Delete All Data
```bash
docker-compose down -v
```

---

## Connection Details

### PostgreSQL
- Host: `localhost`
- Port: `5432`
- User: `analytics_user`
- Password: `analytics_password`
- Database: `analytics_db`

### MongoDB
- Host: `localhost`
- Port: `27017`
- User: `analytics_user`
- Password: `analytics_password`
- URL: `mongodb://analytics_user:analytics_password@localhost:27017/`

---

## Troubleshooting

### Docker Not Running?
```bash
open /Applications/Docker.app
# Wait 30 seconds for Docker Desktop to start
```

### Port Already in Use?
```bash
# Find what's using the port
lsof -i :5432

# Stop old containers
docker stop superset_db
```

### See All Running Containers
```bash
docker ps
```

### See Docker Disk Usage
```bash
docker system df
```

---

## Quick Start Workflow

```bash
# 1. Navigate to project
cd ~/Desktop/data-analytics-dashboard

# 2. Start databases
docker-compose up -d

# 3. Verify running
docker-compose ps

# 4. When done for the day
docker-compose stop
```

---

## What Each Command Does

| Command | What It Does |
|---------|-------------|
| `docker-compose up -d` | Start databases in background |
| `docker-compose stop` | Stop databases (keeps data) |
| `docker-compose down` | Stop and remove containers (keeps data) |
| `docker-compose down -v` | Delete everything including data |
| `docker-compose ps` | Show status of containers |
| `docker-compose logs` | Show container logs |
| `docker-compose restart` | Restart containers |
| `docker-compose exec` | Run command inside container |



To access postgres database in local, 
start docker-compose first
and then use DBeaver application to access database locally.

Credentials.

How to Access Your Data --- refer docker_guide.md file for steps