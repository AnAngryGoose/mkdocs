# Dozzle 

[Github :simple-github: ](https://github.com/amir20/dozzle)

[Official Site :material-web: ](https://dozzle.dev/)

---


Dozzle is a small lightweight application with a web based interface to monitor Docker logs. It doesn’t store any log files. It is for live monitoring of your container logs only.

## Features

- Intelligent fuzzy search for container names 🤖
- Search logs using regex 🔦
- Search logs using [SQL queries](https://dozzle.dev/guide/sql-engine) 📊
- Small memory footprint 🏎
- Split screen for viewing multiple logs
- Live stats with memory and CPU usage
- Multi-user [authentication](https://dozzle.dev/guide/authentication) with support for proxy forward authorization 🚨
- [Swarm](https://dozzle.dev/guide/swarm-mode) mode support 🐳
- [Agent](https://dozzle.dev/guide/agent) mode for monitoring multiple Docker hosts 🕵️‍♂️
- Dark mode 🌙

## Running Dozzle

Dozzle will be available at http://localhost:8080/.

Here is the Docker Compose file:

```yaml
services:
  dozzle:
    container_name: dozzle
    image: amir20/dozzle:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - 8080:8080
```


For advanced options like [authentication](https://dozzle.dev/guide/authentication), [remote hosts](https://dozzle.dev/guide/remote-hosts) or common [questions](https://dozzle.dev/guide/faq) see documentation at [dozzle.dev](https://dozzle.dev/guide/getting-started).


## Remote Hosts

Add the line `command: agent` to your remote host.

```yaml
services:
  dozzle-agent:
    image: amir20/dozzle:latest
    command: agent 
    ports:
      - 7007:7007
    healthcheck:
      test: ["CMD", "/dozzle", "healthcheck"]
      interval: 5s
      retries: 5
      start_period: 5s
      start_interval: 5s
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    restart: unless-stopped
```

On your main host, add the line `command: --remote-agent REMOTEIP:PORT`

```yaml
dozzle:
    image: amir20/dozzle:latest # https://github.com/amir20/dozzle
    container_name: dozzle
    command: --remote-agent 192.168.0.11:7007 # connect to remote agent
    ports:
      - "8081:8080" # Web UI on port 8081
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    environment:
      - DOZZLE_LEVEL=info 
    networks: 
      - proxy_net # to allow npm access (using container name only)
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "/dozzle", "healthcheck"]
      interval: 5s
      retries: 5
      start_period: 5s
      start_interval: 5s
```