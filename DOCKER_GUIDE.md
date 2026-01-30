# 🐳 Docker Guide for Data Analytics Dashboard

A comprehensive guide to understanding Docker, Docker Compose, and how they work in this project.

---

## 📚 Table of Contents

1. [What is Docker?](#what-is-docker)
2. [What is Docker Compose?](#what-is-docker-compose)
3. [How Docker Images Work](#how-docker-images-work)
4. [Understanding Volumes (Data Persistence)](#understanding-volumes-data-persistence)
5. [Our docker-compose.yml Explained](#our-docker-composeyml-explained)
6. [Common Docker Commands](#common-docker-commands)
7. [Data Persistence Scenarios](#data-persistence-scenarios)
8. [How to Access Your Data](#how-to-access-your-data)
9. [Troubleshooting](#troubleshooting)

---

## What is Docker?

**Docker** is a platform that lets you run applications in **containers** - isolated environments that include everything needed to run the application.

### Key Concepts:

- **Image** = Blueprint/Template (read-only)
- **Container** = Running instance created from an image
- **Volume** = Persistent storage that survives container deletion

### Analogy:
```
Image = Recipe (blueprint)
Container = Actual cake baked from recipe
Volume = Refrigerator where you store the cake
```

---

## What is Docker Compose?

**Docker Compose** is a tool for defining and running **multiple Docker containers** together as one application.

Instead of running separate commands for each service:
```bash
docker run postgres
docker run mongodb
docker run backend
```

You define everything in one `docker-compose.yml` file and run:
```bash
docker-compose up -d
```

### Benefits:
- ✅ One command to start everything
- ✅ Services can communicate with each other
- ✅ Consistent environment across teams
- ✅ Easy to version control

---

## How Docker Images Work

### Where Do Images Come From?

Images are downloaded from **Docker Hub** (https://hub.docker.com/) - a public registry of Docker images.

### Download Process:

**First Time:**
```
Docker Hub (Cloud)
    │
    │ docker-compose up (downloads ~480MB)
    ↓
Your Mac (Local Storage)
    ├── postgres:15-alpine (80MB)
    └── mongo:7 (400MB)
```

**Second Time (Fast!):**
```
Your Mac (Uses cached images)
    ├── postgres:15-alpine ✅ Already downloaded
    └── mongo:7 ✅ Already downloaded

Starts in seconds!
```

### Where Are Images Stored?

Images are stored in Docker's internal storage:
```
/Users/surendar/Library/Containers/com.docker.docker/Data/...
```

You don't interact with these files directly - Docker manages them.

### Image vs Container:

```
One Image → Multiple Containers

postgres:15-alpine (IMAGE)
    ├── analytics-postgres (Container 1) ← Running
    ├── test-postgres (Container 2) ← Stopped
    └── dev-postgres (Container 3) ← Running
```

---

## Understanding Volumes (Data Persistence)

### What is a Volume?

A **volume** is Docker's way of storing data **permanently** on your Mac, even when containers are deleted.

### Volume Syntax:

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
       ↓                    ↓
   Named Volume        Path inside container
   (on your Mac)       (where Postgres stores data)
```

### Where is Volume Data Stored?

**Location on your Mac:**
```
/Users/surendar/Library/Containers/com.docker.docker/Data/vms/0/data/docker/volumes/
```

**Specific volume:**
```
.../volumes/data-analytics-dashboard_postgres_data/_data/
```

Docker automatically prefixes volume names with your project directory name.

### Why Volumes Matter:

**Without volumes:**
```
Start container → Add data → Stop container → ❌ Data lost forever
```

**With volumes:**
```
Start container → Add data → Stop container → ✅ Data saved on Mac
Restart container → ✅ Data still there!
Delete container → ✅ Data still there!
```

---

## Our docker-compose.yml Explained

### Full Structure:

```yaml
version: '3.8'                    # Docker Compose file format version

services:                         # Define all services (containers)
  postgres:                       # Service name
    image: postgres:15-alpine     # Docker image to use
    container_name: analytics-postgres  # Container name
    environment:                  # Environment variables
      POSTGRES_USER: analytics_user
      POSTGRES_PASSWORD: analytics_password
      POSTGRES_DB: analytics_db
    ports:                        # Port mapping (host:container)
      - "5432:5432"
    volumes:                      # Persistent storage
      - postgres_data:/var/lib/postgresql/data
    healthcheck:                  # Health monitoring
      test: ["CMD-SHELL", "pg_isready -U analytics_user"]
      interval: 10s
      timeout: 5s
      retries: 5

  mongodb:
    image: mongo:7
    container_name: analytics-mongo
    environment:
      MONGO_INITDB_ROOT_USERNAME: analytics_user
      MONGO_INITDB_ROOT_PASSWORD: analytics_password
      MONGO_INITDB_DATABASE: analytics_db
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:                          # Define named volumes
  postgres_data:                  # Managed by Docker
  mongo_data:                     # Managed by Docker
```

### Line-by-Line Breakdown:

#### PostgreSQL Service

| Line | What It Does |
|------|--------------|
| `image: postgres:15-alpine` | Download PostgreSQL 15 (Alpine = small version) |
| `container_name: analytics-postgres` | Name the container for easy identification |
| `POSTGRES_USER: analytics_user` | Create database user on first startup |
| `POSTGRES_PASSWORD: analytics_password` | Set password (⚠️ change in production!) |
| `POSTGRES_DB: analytics_db` | Create database on first startup |
| `ports: "5432:5432"` | Map port 5432 (Mac) ↔ 5432 (container) |
| `postgres_data:/var/lib/postgresql/data` | Store database files permanently on Mac |
| `healthcheck` | Check every 10s if database is ready |

#### MongoDB Service

| Line | What It Does |
|------|--------------|
| `image: mongo:7` | Download MongoDB version 7 |
| `ports: "27017:27017"` | Map port 27017 (Mac) ↔ 27017 (container) |
| `mongo_data:/data/db` | Store MongoDB files permanently on Mac |

#### Environment Variables

These are passed to containers on startup:
- PostgreSQL reads `POSTGRES_*` variables to configure itself
- MongoDB reads `MONGO_*` variables
- Your FastAPI backend will use these same credentials to connect

#### Port Mapping

```
"5432:5432"
  ↓     ↓
 Mac   Container
```

- **Left (5432):** Port on your Mac
- **Right (5432):** Port inside container
- You connect to `localhost:5432` on your Mac
- Traffic is forwarded to container's port 5432

#### Healthcheck

```yaml
healthcheck:
  test: ["CMD-SHELL", "pg_isready -U analytics_user"]
  interval: 10s
  timeout: 5s
  retries: 5
```

- Runs `pg_isready` command every 10 seconds
- If it succeeds → Container is "healthy" ✅
- If it fails 5 times → Container is "unhealthy" ❌
- Other services can wait for this to be healthy before starting

---

## Common Docker Commands

### Starting Services

```bash
# Start all services in background
docker-compose up -d

# Start and view logs
docker-compose up

# Start specific service only
docker-compose up -d postgres
```

### Stopping Services

```bash
# Stop all services (keeps containers)
docker-compose stop

# Stop specific service
docker-compose stop postgres

# Stop and remove containers (⚠️ keeps volumes)
docker-compose down

# Stop, remove containers AND volumes (⚠️⚠️ deletes all data)
docker-compose down -v
```

### Viewing Status

```bash
# List running containers
docker-compose ps

# View logs (all services)
docker-compose logs

# View logs (specific service)
docker-compose logs postgres

# Follow logs (live)
docker-compose logs -f

# View last 50 lines
docker-compose logs --tail=50
```

### Managing Containers

```bash
# Restart all services
docker-compose restart

# Restart specific service
docker-compose restart postgres

# Execute command in running container
docker-compose exec postgres psql -U analytics_user -d analytics_db

# Access container shell
docker-compose exec postgres bash
```

### Managing Images

```bash
# List downloaded images
docker images

# Remove specific image
docker rmi postgres:15-alpine

# Pull latest images without starting
docker-compose pull
```

### Managing Volumes

```bash
# List all volumes
docker volume ls

# Inspect volume (see location and size)
docker volume inspect data-analytics-dashboard_postgres_data

# Remove specific volume (⚠️ deletes data)
docker volume rm data-analytics-dashboard_postgres_data

# Remove all unused volumes (⚠️ dangerous)
docker volume prune
```

### System Cleanup

```bash
# View disk usage
docker system df

# View detailed disk usage (including volumes)
docker system df -v

# Remove all unused containers, networks, images
docker system prune

# Remove everything including volumes (⚠️⚠️ nuclear option)
docker system prune -a --volumes
```

---

## Data Persistence Scenarios

### Scenario 1: Normal Workflow (Data Persists)

```bash
# Day 1: Start fresh
$ docker-compose up -d
$ # Upload 100 CSV files, create 20 SQL queries
$ docker-compose down          # ✅ Data saved in volumes

# Day 2: Resume work
$ docker-compose up -d         # ✅ All 100 files and 20 queries still there!

# Day 3: Stop for the day
$ docker-compose stop          # ✅ Data safe

# Day 4: Continue
$ docker-compose start         # ✅ Everything intact
```

### Scenario 2: Fresh Start (Delete All Data)

```bash
# Current state: Database has old test data
$ docker-compose down -v       # ⚠️ Deletes all data

# Start with clean slate
$ docker-compose up -d         # Empty databases
```

### Scenario 3: Backup Before Destructive Action

```bash
# Backup before deleting
$ docker-compose exec postgres pg_dump -U analytics_user analytics_db > backup.sql
$ docker-compose down -v       # ⚠️ Delete everything

# Restore later
$ docker-compose up -d
$ cat backup.sql | docker-compose exec -T postgres psql -U analytics_user -d analytics_db
```

### Visual Timeline

| Time | Command | Containers | Volumes | Your Data |
|------|---------|------------|---------|-----------|
| T1 | `docker-compose up -d` | Running ✅ | Created ✅ | Empty |
| T2 | Upload 100 files | Running ✅ | Exist ✅ | 100 files ✅ |
| T3 | `docker-compose stop` | Stopped 🛑 | Exist ✅ | **100 files ✅** |
| T4 | `docker-compose start` | Running ✅ | Exist ✅ | **100 files ✅** |
| T5 | `docker-compose down` | Deleted ❌ | Exist ✅ | **100 files ✅** |
| T6 | `docker-compose up -d` | Running ✅ | Exist ✅ | **100 files ✅** |
| T7 | `docker-compose down -v` | Deleted ❌ | Deleted ❌ | **GONE ❌** |

---

## How to Access Your Data

### Method 1: Database CLI Tools

**PostgreSQL:**
```bash
# Connect to PostgreSQL (while containers are running)
psql -h localhost -p 5432 -U analytics_user -d analytics_db

# Or using Docker:
docker-compose exec postgres psql -U analytics_user -d analytics_db
```

**MongoDB:**
```bash
# Connect to MongoDB
mongosh mongodb://analytics_user:analytics_password@localhost:27017/

# Or using Docker:
docker-compose exec mongodb mongosh -u analytics_user -p analytics_password
```

### Method 2: GUI Database Tools (Recommended for Beginners)

#### DBeaver for PostgreSQL

**Step 1: Install DBeaver**
- Download from: https://dbeaver.io/download/
- Or install via Homebrew: `brew install --cask dbeaver-community`

**Step 2: Open DBeaver and Create Connection**
1. Click **Database** → **New Database Connection**
2. Or click the **plug icon** with a green plus sign in the toolbar

**Step 3: Select PostgreSQL**
1. Find and click **PostgreSQL** in the list
2. Click **Next**

**Step 4: Enter Connection Details**
```
Host:      localhost
Port:      5432
Database:  analytics_db
Username:  analytics_user
Password:  analytics_password
```

**Step 5: Test Connection**
1. Click **Test Connection** button
2. If first time, DBeaver will ask to download the PostgreSQL driver
3. Click **Download** and wait for it to complete
4. Should show "Connected" message ✅
5. Click **Finish**

**Step 6: Explore Your Data**
1. In the left panel, expand the connection tree:
   - `analytics_db` → `Schemas` → `public` → `Tables`
2. You'll see `test_persistence` table (from our test)
3. Right-click the table → **View Data**
4. You should see the 2 test rows we created earlier

**Common DBeaver Actions:**
- **View data:** Right-click table → View Data
- **Run SQL query:** Click SQL Editor icon (or press F3)
- **Export data:** Right-click table → Export Data

---

#### MongoDB Compass for MongoDB

**Step 1: Install MongoDB Compass**
- Download from: https://www.mongodb.com/try/download/compass
- Or install via Homebrew: `brew install --cask mongodb-compass`

**Step 2: Open MongoDB Compass**

**Step 3: Create New Connection**
1. Click **New Connection** button (or the green "Connect" button)

**Step 4: Enter Connection String**

**Option A: Use Connection String (Easiest)**

Paste this entire string in the connection box:
```
mongodb://analytics_user:analytics_password@localhost:27017/
```

**Option B: Use Advanced Connection Form**
1. Click **"Fill in connection fields individually"**
2. Enter:
   ```
   Host:                    localhost
   Port:                    27017
   Authentication:          Username/Password
   Username:                analytics_user
   Password:                analytics_password
   Authentication Database: admin
   ```

**Step 5: Save and Connect**
1. Click **Save & Connect**
2. Give it a name: "Local Analytics DB"
3. Click **Connect**

**Step 6: Explore Your Data**
1. You'll see default databases: `admin`, `config`, `local`
2. Currently no custom data (we haven't used MongoDB yet in our app)
3. You can create a test database:
   - Click **"Create Database"**
   - Database name: `test_db`
   - Collection name: `test_collection`
   - Click **Create Database**

**Common MongoDB Compass Actions:**
- **View collections:** Click database → click collection
- **Insert document:** Click "Add Data" → Insert Document
- **Query data:** Use the filter bar at the top

---

#### Quick Connection Reference

**PostgreSQL (DBeaver):**
```
Host: localhost
Port: 5432
User: analytics_user
Password: analytics_password
Database: analytics_db
```

**MongoDB (Compass):**
```
Connection String:
mongodb://analytics_user:analytics_password@localhost:27017/

OR

Host: localhost:27017
Username: analytics_user
Password: analytics_password
Auth Database: admin
```

---

#### Troubleshooting GUI Connections

**"Connection refused" error?**
1. Make sure Docker containers are running:
   ```bash
   cd ~/Desktop/data-analytics-dashboard
   docker-compose ps
   ```
2. Should show both containers as "Running"
3. If not:
   ```bash
   docker-compose up -d
   ```

**DBeaver: "Driver not found" error?**
1. When testing connection, click **Download** when prompted
2. DBeaver will automatically download the PostgreSQL driver
3. Try connection again

**MongoDB Compass: "Authentication failed"?**
1. Make sure you're using `admin` as the Authentication Database
2. Double-check username: `analytics_user` (not `admin`)
3. Double-check password: `analytics_password`

**Can't see data in DBeaver?**
1. Make sure you're looking in the right place:
   - Connection → `analytics_db` → `Schemas` → `public` → `Tables`
2. Refresh the database tree (right-click → Refresh)

---

#### Alternative GUI Tools

**Other PostgreSQL Tools:**
- [pgAdmin](https://www.pgadmin.org/) - Free, feature-rich, web-based
- [TablePlus](https://tableplus.com/) - Beautiful, Mac-native ($$$, free tier available)
- [Postico](https://eggerapps.at/postico/) - Mac-only, simple and elegant

**Other MongoDB Tools:**
- [Studio 3T](https://studio3t.com/) - Advanced features, free trial
- [Robo 3T](https://robomongo.org/) - Lightweight, free

### Method 3: Backup/Export Data

**PostgreSQL Backup:**
```bash
# Export entire database
docker-compose exec postgres pg_dump -U analytics_user analytics_db > backup.sql

# Export specific table
docker-compose exec postgres pg_dump -U analytics_user -d analytics_db -t users > users_backup.sql

# Restore from backup
cat backup.sql | docker-compose exec -T postgres psql -U analytics_user -d analytics_db
```

**MongoDB Backup:**
```bash
# Export entire database
docker-compose exec mongodb mongodump --username analytics_user --password analytics_password --out /backup

# Copy backup from container to Mac
docker cp analytics-mongo:/backup ./mongodb_backup

# Restore from backup
docker-compose exec mongodb mongorestore --username analytics_user --password analytics_password /backup
```

### Method 4: Direct Volume Access (Advanced)

**⚠️ Not recommended for beginners**

Volumes are stored in Docker's VM and use database-specific binary formats. Better to use methods above.

---

## Troubleshooting

### Issue: "Cannot connect to the Docker daemon"

```
Error: Cannot connect to the Docker daemon at unix:///var/run/docker.sock
```

**Solution:**
1. Check if Docker Desktop is running (whale icon in menu bar)
2. If not, open Docker Desktop: `/Applications/Docker.app`
3. Wait for it to fully start (30-60 seconds)
4. Try again

### Issue: "Port 5432 is already in use"

```
Error: Bind for 0.0.0.0:5432 failed: port is already allocated
```

**Solution:**
```bash
# Check what's using port 5432
lsof -i :5432

# If PostgreSQL is installed locally, stop it:
brew services stop postgresql

# Or change port in docker-compose.yml:
ports:
  - "5433:5432"  # Use 5433 on Mac instead
```

### Issue: "Volume is in use and cannot be removed"

```
Error: volume is in use
```

**Solution:**
```bash
# Stop all containers first
docker-compose down

# Then remove volume
docker volume rm data-analytics-dashboard_postgres_data
```

### Issue: Database connection refused

**Solution:**
```bash
# Check if containers are running
docker-compose ps

# Check container logs
docker-compose logs postgres
docker-compose logs mongodb

# Restart services
docker-compose restart
```

### Issue: Out of disk space

```bash
# Check Docker disk usage
docker system df

# Clean up unused images and containers
docker system prune

# Remove unused volumes (⚠️ may delete data)
docker volume prune
```

### Issue: Containers start but immediately stop

```bash
# View logs to see error
docker-compose logs

# Common causes:
# - Port already in use
# - Invalid environment variables
# - Insufficient memory
```

---

## Best Practices

### Development Workflow

1. **Start of day:**
   ```bash
   cd ~/Desktop/data-analytics-dashboard
   docker-compose up -d
   ```

2. **End of day:**
   ```bash
   docker-compose stop  # Keeps data safe
   ```

3. **Need fresh start:**
   ```bash
   # Backup first if needed
   docker-compose exec postgres pg_dump -U analytics_user analytics_db > backup.sql

   # Then reset
   docker-compose down -v
   docker-compose up -d
   ```

### Security Tips

- ⚠️ **Never commit** `docker-compose.yml` with real passwords to public repos
- ✅ Use environment variables for production
- ✅ Change default passwords in production
- ✅ Use `.env` files (add to `.gitignore`)

**Example `.env` file:**
```bash
POSTGRES_USER=analytics_user
POSTGRES_PASSWORD=super_secure_password_here
POSTGRES_DB=analytics_db
```

**Reference in docker-compose.yml:**
```yaml
environment:
  POSTGRES_USER: ${POSTGRES_USER}
  POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
  POSTGRES_DB: ${POSTGRES_DB}
```

### Performance Tips

```bash
# Limit log size to prevent disk space issues
docker-compose logs --tail=100

# Set log rotation in docker-compose.yml:
logging:
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"
```

---

## Quick Reference Card

| Task | Command |
|------|---------|
| **Start everything** | `docker-compose up -d` |
| **Stop everything (keep data)** | `docker-compose stop` |
| **Stop and remove containers (keep data)** | `docker-compose down` |
| **Delete everything including data** | `docker-compose down -v` |
| **View logs** | `docker-compose logs -f` |
| **Check status** | `docker-compose ps` |
| **Restart service** | `docker-compose restart postgres` |
| **Connect to PostgreSQL** | `docker-compose exec postgres psql -U analytics_user -d analytics_db` |
| **Connect to MongoDB** | `docker-compose exec mongodb mongosh` |
| **Backup PostgreSQL** | `docker-compose exec postgres pg_dump -U analytics_user analytics_db > backup.sql` |
| **List volumes** | `docker volume ls` |
| **Check disk usage** | `docker system df` |

---

## Next Steps

Now that you understand Docker:

1. ✅ Start Docker Desktop
2. ✅ Run `docker-compose up -d`
3. ✅ Verify databases are running: `docker-compose ps`
4. ✅ Connect with database client to test
5. ✅ Move on to building the FastAPI backend!

---

## Additional Resources

- [Docker Documentation](https://docs.docker.com/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [PostgreSQL Docker Hub](https://hub.docker.com/_/postgres)
- [MongoDB Docker Hub](https://hub.docker.com/_/mongo)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)

---

**Questions?** Refer to the [Troubleshooting](#troubleshooting) section or check the official documentation.
