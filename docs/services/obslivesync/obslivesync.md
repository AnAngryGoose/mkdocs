# Self-Hosted Obsidian LiveSync with Tailscale

---

This setup provides a self-hosted synchronization backend for Obsidian using CouchDB. It is configured to run behind Tailscale for secure remote access without exposing ports to the public internet.

## 1\. Docker Compose Configuration

The service runs as a specific user (`5984`) to match the internal CouchDB user ID.

**`compose.yaml`**

```yaml
services:
  couchdb:
    image: couchdb:latest
    container_name: couchdb-for-ols
    user: 5984:5984
    environment:
      - COUCHDB_USER=admin  # Change strictly in .env or secrets for production
      - COUCHDB_PASSWORD=password # Change strictly in .env or secrets for production
    volumes:
      - ./couchdb-data:/opt/couchdb/data
      - ./couchdb-etc:/opt/couchdb/etc/local.d
    ports:
      - 5984:5984
    restart: unless-stopped
```

## 2\. Permission Management (User 5984)

The official CouchDB image runs as UID/GID `5984`. Since Docker mounts host directories (`./couchdb-data`) with root or current user permissions by default, the container will fail with `Permission denied` when attempting to write configuration files.

**The Fix:**
Pre-create directories and set strict ownership to UID `5984` before starting the container.

```bash
mkdir -p ./couchdb-data ./couchdb-etc
sudo chown -R 5984:5984 ./couchdb-data
sudo chown -R 5984:5984 ./couchdb-etc
```

*Note: If you need to edit files in `couchdb-etc` manually later, you must temporarily `chown` them back to your user or edit as root.*

## 3\. CouchDB Configuration (`docker.ini`)

This configuration is injected via the `./couchdb-etc` volume. It handles Large File Support (LFS), legacy authentication, and critical CORS headers for remote sync.

**File Location:** `./couchdb-etc/docker.ini`

```ini
[couchdb]
single_node = true
max_document_size = 50000000

[chttpd]
bind_address = 0.0.0.0
port = 5984
require_valid_user = true
max_http_request_size = 4294967296
enable_cors = true

[chttpd_auth]
require_valid_user = true
authentication_redirect = /_utils/session.html

[httpd]
WWW-Authenticate = Basic realm="couchdb"
enable_cors = true

[cors]
# Wildcard required for Tailscale/VPN roaming where Client IPs change
origins = *
credentials = true
# Explicit headers required for Obsidian LiveSync Plugin PUT/Auth operations
headers = accept, authorization, content-type, origin, referer
methods = GET, PUT, POST, HEAD, DELETE
max_age = 3600
```

### Configuration Notes

  * **`origins = *`**: Essential for Tailscale. Strict origin validation often fails because the "Origin" header sent by mobile WebViews inside a VPN tunnel may not match the server's expected hostname.
  * **`methods = ...`**: The standard pre-flight check fails without explicitly allowing `PUT` (for saving notes) and `DELETE`.
  * **`max_http_request_size`**: Increased to \~4GB to prevent sync failures on vaults containing large binaries.

## 3.1 Run couchdb-init.sh for initialise
```
curl -s https://raw.githubusercontent.com/vrtmrz/obsidian-livesync/main/utils/couchdb/couchdb-init.sh | bash
```

If it results like the following:
```
-- Configuring CouchDB by REST APIs... -->
{"ok":true}
""
""
""
""
""
""
""
""
""
<-- Configuring CouchDB by REST APIs Done!
```

Your CouchDB has been initialised successfully. If you want this manually, please read the script.

If you are using Docker Compose and the above command does not work or displays `ERROR: Hostname missing`, you can try running the following command, replacing the placeholders with your own values:
```
curl -s https://raw.githubusercontent.com/vrtmrz/obsidian-livesync/main/utils/couchdb/couchdb-init.sh | hostname=http://<YOUR SERVER IP>:5984 username=<INSERT USERNAME HERE> password=<INSERT PASSWORD HERE> bash
```

## 4\. Generate the setup URI on a desktop device or server
```bash
export hostname=https://you.tailscale.ip:5984 #Use tailscale IP
export database=obsidiannotes 
export passphrase=dfsapkdjaskdjasdas #This is for the encryption
export username=johndoe
export password=abc123 #this is your couchdb password in compose.yaml
deno run -A https://raw.githubusercontent.com/vrtmrz/obsidian-livesync/main/utils/flyio/generate_setupuri.ts
```
You will then get the following output:

```bash
obsidian://setuplivesync?settings=%5B%22tm2DpsOE74nJAryprZO2M93wF%2Fvg.......4b26ed33230729%22%5D

Your passphrase of Setup-URI is:  patient-haze
This passphrase is never shown again, so please note it in a safe place.
```

Keep a copy of this Setup-URI. Not gonna come back
## 4\. Tailscale & Network Implementation

### Connection String

Mobile OSs often block cleartext HTTP traffic to **hostnames**, even over a VPN. Using this when connected to tailscale only will enable you to safely and easily bypass the required HTTPS when syncing on mobile.

  * **Do not use:** `http://my-server-name:5984`
  * **Use:** `http://100.x.y.z:5984` (The specific Tailscale IP of the host).

### Android Optimization (optional)

To prevent "Silent Drift" (sync stopping in background):

1.  **Battery:** Set Obsidian app to **Unrestricted** / No Optimization.
2.  **Plugin Settings:** Manually switch Android clients to "Periodic" (30s) or "Adaptive".

## **<ins>_CRITICAL WARNING: Customization Sync_</ins>**

During or after setup, the plugin may ask to enable **"Customization Sync"** or "Configuration Sync" (synchronizing plugins, themes, and settings).

  * **DO NOT ENABLE THIS. THIS WILL MAKE USING LIVESYNC A FUCKING NIGHTMARE**
  * **Decline all pop-ups** related to syncing configuration files or hidden folders (`.obsidian`).

**Reasoning:** Syncing configuration files between different platforms (e.g., Windows and Android) frequently causes boot loops, UI corruption, and file path conflicts. Sync **only** your notes (markdown) and media.

## 5\. Initial Client Setup (Primary Device)


Perform these steps on the **first** device to initialize the database structure.

1.  Select **"I am setting this up for the first time"**.
2.  Input the **Setup URI** generated/configured previously.
3.  Select **"Restart and initialize server"**.
4.  On the final confirmation/overwrite page: Select **YES** on all prompts.
5.  On the "Fetch remote config failed" page (Expected, as database is empty): Select **"Skip and Proceed"**.
6.  On the "Send all Chunks" page: Select **NO**.
7.  On the "Config Doctor" page: Select **NO** on all prompts.
8.  Under Sync Settings, enable the LiveSync preset configuration (on all devices)

## 6\. Adding Additional Clients

Perform these steps when connecting secondary devices (Mobile/Laptop) to the existing database.

1.  Enter the **Setup URI** and **Passphrase**.
2.  Select **"Restart and fetch data"**.
3.  On the "Reset Synchronisation on This Device" page: Select the setting applicable to your needs (e.g., Merge or Discard Local).
4.  On the "All optional Features are disabled" page: Select **OK**.
5.  On the "Config Doctor" page: Select **NO** on all prompts.
6. Under Sync Settings, enable the LiveSync preset configuration (on all devices)
