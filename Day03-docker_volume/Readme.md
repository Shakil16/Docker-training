# Docker Volumes & Persistent Storage

Containers are **ephemeral** by default. Any data written inside a container is lost when the container is removed.

Docker provides **Volumes** and **Bind Mounts** to persist data outside the container lifecycle.

---

# List Existing Volumes

```bash
docker volume ls
```

Example:

```text
DRIVER    VOLUME NAME
local     mongodb
local     portainer_data
```

---

# Create a Docker Volume

```bash
docker volume create mongodb
```

Verify:

```bash
docker volume ls
```

---

# Run MongoDB Using a Docker Volume

```bash
docker run \
-d \
--rm \
--name mongodb \
-v mongodb:/data/db \
-p 27017:27017 \
mongo:latest
```

### Explanation

| Option | Description |
|---------|-------------|
| `-d` | Run container in background |
| `--rm` | Remove container when stopped |
| `--name mongodb` | Container name |
| `-v mongodb:/data/db` | Mount Docker volume |
| `-p 27017:27017` | Expose MongoDB port |
| `mongo:latest` | MongoDB image |

---

# Verify Container

```bash
docker ps
```

Expected output:

```text
CONTAINER ID   IMAGE          STATUS
abc123         mongo:latest   Up 2 minutes
```

---

# Connect to MongoDB

```bash
docker exec -it mongodb mongosh
```

Show databases:

```javascript
show dbs
```

Create or switch database:

```javascript
use demo
```

---

# Insert Sample Data

Create a collection named **helo**.

```javascript
db.helo.insertMany([
  {
    "_id": 1,
    "name": "Matt",
    "status": "active",
    "level": 12,
    "score": 202
  },
  {
    "_id": 2,
    "name": "Frank",
    "status": "inactive",
    "level": 2,
    "score": 9
  },
  {
    "_id": 3,
    "name": "Karen",
    "status": "active",
    "level": 7,
    "score": 87
  },
  {
    "_id": 4,
    "name": "Katie",
    "status": "married",
    "level": 3,
    "score": 27,
    "emp": "yes",
    "kids": 3
  }
])
```

Query the collection:

```javascript
db.helo.find({ name: "Katie" })
```

Return all documents:

```javascript
db.helo.find()
```

Pretty output:

```javascript
db.helo.find().pretty()
```

---

# Insert Sample Bios Collection

Create another collection named **bios**.

```javascript
db.bios.insertMany([
  {
    name: {
      first: "John",
      last: "Backus"
    },
    contribs: [
      "Fortran",
      "ALGOL",
      "FP"
    ]
  },
  {
    name: {
      first: "Grace",
      last: "Hopper"
    },
    contribs: [
      "COBOL",
      "Compiler"
    ]
  },
  {
    name: {
      first: "Guido",
      last: "van Rossum"
    },
    contribs: [
      "Python"
    ]
  }
])
```

Example query:

```javascript
db.bios.find()
```

Find Python contributor:

```javascript
db.bios.find({
    contribs: "Python"
})
```

---

# Run Troubleshooting Tools Container

```bash
docker run \
-d \
--rm \
--name app1 \
-v /var/run/docker.sock:/var/run/docker.sock \
--network none \
shakil1602/troubleshootingtools:v1
```

## Explanation

| Option | Description |
|---------|-------------|
| `--network none` | Disable networking |
| `/var/run/docker.sock` | Allows the container to communicate with Docker daemon |
| `--rm` | Remove container automatically |

---

# Portainer

Portainer is a lightweight web UI for managing Docker environments.

---

## Create Volume

```bash
docker volume create portainer_data
```

---

## Run Portainer

```bash
docker run -d \
-p 8000:8000 \
-p 9443:9443 \
--name portainer \
--restart always \
-v /var/run/docker.sock:/var/run/docker.sock \
-v portainer_data:/data \
portainer/portainer-ce:2.11.1
```

---

## Command Breakdown

| Option | Description |
|---------|-------------|
| `-d` | Detached mode |
| `-p 8000:8000` | Portainer Agent |
| `-p 9443:9443` | HTTPS Web UI |
| `--restart always` | Automatically restart container |
| `-v /var/run/docker.sock:/var/run/docker.sock` | Access Docker daemon |
| `-v portainer_data:/data` | Persistent Portainer data |
| `portainer/portainer-ce:2.11.1` | Docker image |

---

## Access Portainer

Open your browser:

```
https://<SERVER-IP>:9443
```

Example:

```
https://54.71.74.129:9443
```

The first time you log in, Portainer will ask you to:

- Create an admin user
- Configure the local Docker environment

---

# Container Storage Internals

Every container consists of:

```
           Writable Layer
        (Container Layer)
                ▲
────────────────────────────────
        Read-only Image Layer 3
────────────────────────────────
        Read-only Image Layer 2
────────────────────────────────
        Read-only Image Layer 1
────────────────────────────────
```

The **Writable Layer** stores:

- New files
- Modified files
- Deleted files

If the container is removed, **this writable layer is also removed**.

Therefore, important data should never be stored only inside the container.

---

# Docker Storage Options

| Storage Type | Description | Best Use |
|--------------|-------------|----------|
| Volume | Docker-managed persistent storage | Databases, application data |
| Bind Mount | Mounts a host directory | Development |
| tmpfs | Memory-backed filesystem | Temporary or sensitive data |

---

# Docker Volumes

Docker manages the storage location.

Create a volume:

```bash
docker volume create pgdata
```

Use it:

```bash
docker run \
-v pgdata:/var/lib/postgresql/data \
postgres:16
```

Advantages:

- Persistent
- Portable
- Docker-managed
- Recommended for databases

---

# Bind Mounts

Bind mounts expose an existing directory from the host.

Linux example:

```bash
docker run \
-v /home/user/app:/app \
node:20
```

Windows example:

```powershell
docker run `
-v C:\app:/app `
node:20
```

Advantages:

- Easy code editing
- Ideal for development
- Changes are immediately reflected inside the container

---

# tmpfs Mounts

tmpfs stores data **only in memory**.

```bash
docker run \
--tmpfs /app/tmp \
nginx
```

Characteristics:

- Fast
- No disk writes
- Data disappears when the container stops

Ideal for:

- Temporary files
- Cache
- Secrets
- Sensitive information

---

# Volume vs Bind Mount vs tmpfs

| Feature | Volume | Bind Mount | tmpfs |
|---------|---------|------------|--------|
| Persistent | ✅ | ✅ | ❌ |
| Docker Managed | ✅ | ❌ | ❌ |
| Uses Host Directory | ❌ | ✅ | ❌ |
| Uses RAM | ❌ | ❌ | ✅ |
| Best For | Databases | Development | Temporary Data |

---

# Inspect Docker Volumes

List volumes:

```bash
docker volume ls
```

Inspect a volume:

```bash
docker volume inspect mongodb
```

Example output:

```json
[
  {
    "Name": "mongodb",
    "Driver": "local",
    "Mountpoint": "/var/lib/docker/volumes/mongodb/_data"
  }
]
```

---

# Remove Volumes

Remove one volume:

```bash
docker volume rm mongodb
```

Remove unused volumes:

```bash
docker volume prune
```

---

# Best Practices

- ✅ Use **Volumes** for databases (MongoDB, PostgreSQL, MySQL, Redis).
- ✅ Use **Bind Mounts** during local development.
- ✅ Use **tmpfs** for temporary or sensitive data.
- ❌ Avoid storing important data in a container's writable layer.
- ✅ Inspect volumes regularly using `docker volume inspect`.
- ✅ Back up Docker volumes before deleting them.