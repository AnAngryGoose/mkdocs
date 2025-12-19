---
tags:
  - docker
  - service
  - cheat_sheet
---
___

## 5. Command Reference

### 5.1. Project Management (Docker Compose)

_Primary method for managing the stacks on the M920q._

| **Command**                                                        | **Description**                                                                                                |
| ------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| `docker compose up -d`                                             | Create/Start services in detached mode.                                                                        |
| `docker compose up -d --force-recreate`                            | **Force** container recreation (useful if config changed but image didn't).                                    |
| `docker compose restart`                                           | Stop then restart containers. Does not rebuild/pull updated images.                                            |
| `docker compose down`                                              | Stop and remove containers/networks.                                                                           |
| `docker compose down -v`                                           | **DANGER:** Removes containers AND **standard volumes**.                                                       |
| <br>`docker compose down --volumes --rmi all --remove-orphans`<br> | **DANGER:** Removes all volumes, images built by container (not just this project), and orphaned compose files |
| `docker compose stop`                                              | Stops services without removing containers.                                                                    |
| `docker compose pull`                                              | Downloads latest images defined in YAML.                                                                       |
| `docker compose logs -f`                                           | Stream logs for all services in the stack.                                                                     |
| `docker compose config`                                            | Validate YAML syntax and print the resolved configuration.                                                     |
| `docker compose exec <svc> <cmd>`                                  | Run command inside service (e.g., `docker compose exec pihole bash`).                                          |

### 5.2. Container & Resource Monitoring

_Debugging individual containers or checking Pi 4B resource load._

|**Command**|**Description**|
|---|---|
|`docker ps`|List running containers.|
|`docker ps -a`|List **all** containers (including stopped/exited).|
|`docker stats`|**Live stream** of CPU, RAM, and Net I/O usage per container.|
|`docker logs -f <container>`|Follow log output for a specific container (e.g., `omada-controller`).|
|`docker inspect <container>`|JSON dump of container config (IP address, mounts, env vars).|
|`docker top <container>`|Display the running processes of a container.|

### 5.3. Storage & Volumes

_Managing persistence. Critical for the "External Volume" workflow._

|**Command**|**Description**|
|---|---|
|`docker volume ls`|List all volumes.|
|`docker volume create <name>`|Manually create a volume (required for `external: true`).|
|`docker volume inspect <name>`|Show volume mount point (usually `/var/lib/docker/volumes/...`).|
|`docker volume rm <name>`|Delete a volume. **Data loss permanent.**|
|`docker volume prune`|Remove **all** unused local volumes.|
|`docker run --rm -v <vol>:/data busybox ls /data`|Quickly list files inside a volume without attaching to the main app.|

### 5.4. Networking

_Essential for debugging Pi-hole/Omada communication issues._

|**Command**|**Description**|
|---|---|
|`docker network ls`|List all created networks.|
|`docker network inspect <net_name>`|Show containers and IP addresses assigned to a specific network.|
|`docker network create <name>`|Create a manual bridge network.|
|`docker network prune`|Remove all unused networks.|

### 5.5. System Maintenance (Housekeeping)

_Periodic cleanup for the M920q and Pi 4B._

|**Command**|**Description**|
|---|---|
|`docker image prune -a`|Remove all unused images (frees disk space).|
|`docker system df`|Show docker disk usage summary.|
|`docker system prune`|**Cleanup:** Removes stopped containers, unused networks, and dangling images.|
|`docker system prune -a --volumes`|**Nuclear:** Wipes everything not currently running.|

---

## 6. Workflows

### 6.1. Deploying New Stack (External Volume Method)

_Standard procedure for stateful services (Omada, Databases)._

1. **Pre-create Volumes:**
    
    Bash
    
    ```
    docker volume create service_data
    docker volume create service_config
    ```
    
2. **Deploy:**
    
    Bash
    
    ```
    # Navigate to project folder
    docker compose up -d
    ```
    
3. **Verify Persistence:**
    
    Bash
    
    ```
    docker volume inspect service_data
    ```
    

### 6.2. Updating a Stack (Zero-Downtime Attempt)

_Routine update cycle for the M920q._

Bash

```
cd /opt/docker/project_name

# 1. Pull updates
docker compose pull

# 2. Recreate only changed containers
docker compose up -d

# 3. Clean up old images to save space
docker image prune -f
```

### 6.3. "Hot" Backup of a Volume

_Backup data without stopping the container (using `tar`)._

Bash

```
# Syntax: docker run --rm -v <volume_name>:/data -v <host_backup_dir>:/backup alpine tar czf /backup/<filename>.tar.gz -C /data .

docker run --rm \
  -v omada_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/omada_data_backup_$(date +%F).tar.gz -C /data .
```

### 6.4. Debugging Connectivity

_If Pi-hole or Unbound isn't responding._

Bash

```
# 1. Check if container is healthy
docker ps 

# 2. Check internal IP assignment
docker network inspect bridge_network_name

# 3. Test resolution from INSIDE the container
docker exec -it pihole nslookup google.com 127.0.0.1
```

## 7. Environment Variables (`.env`)

Separates configuration (passwords, versions, ports) from the code (`compose.yaml`).

### 7.1. Directory Location

The `.env` file **must** be located in the same directory as your `compose.yaml` file for Docker Compose to automatically read it.

**File Structure:**

Plaintext

```
~/homelab/docker/
├── .gitignore             # Global git ignore
├── core/                  # Project Stack (e.g., Homepage, Glances)
│   ├── .env               # <--- GOES HERE (Specific to Core)
│   └── compose.yaml
├── omada/                 # Project Stack (Network)
│   ├── .env               # <--- GOES HERE (Specific to Omada)
│   └── compose.yaml
└── couchdb/               # Project Stack (Obsidian Sync)
    ├── .env               # <--- GOES HERE (Specific to CouchDB)
    └── compose.yaml
```

### 7.2. Syntax Rules

- **Format:** `KEY=VALUE`
    
- **No Spaces:** `PASSWORD = 123` is **invalid**. Use `PASSWORD=123`.
    
- **Comments:** Use `#` for comments.
    
- **Special Characters:** If a password contains `#`, `$`, or spaces, wrap the value in quotes: `DB_PASS="Secure#Pass$123"`.
    

### 7.3. Workflow Example (CouchDB)

1. Create the file:

nano ~/homelab/docker/couchdb/.env

**2. Define Variables:**

Ini, TOML

```
# User Configuration
COUCH_USER=admin
COUCH_PASS=MySecretPassword123!

# Network Configuration
HOST_PORT=5984
```

3. Implement in compose.yaml:

Use the ${VARIABLE_NAME} syntax.

YAML

```
services:
  couchdb:
    image: couchdb:latest
    environment:
      - COUCHDB_USER=${COUCH_USER}      # Pulls 'admin'
      - COUCHDB_PASSWORD=${COUCH_PASS}  # Pulls 'MySecretPassword123!'
    ports:
      - ${HOST_PORT}:5984               # Maps port 5984
```

### 7.4. Verification

To verify that Docker is reading the variables correctly without actually starting the container, run:

Bash

```
docker compose config
```

_This prints the "resolved" YAML to the terminal, showing the actual values instead of the `${VAR}` placeholders._

### 7.5. Security (Git)

NEVER commit .env files to GitHub. They contain your secrets.

Ensure your global .gitignore includes:

Plaintext

```
**/.env
**/.env.*
```