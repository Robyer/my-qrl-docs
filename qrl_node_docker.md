# QRL Node Docker Setup Guide

Guide for installing and configuring a QRL blockchain node using Docker.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Install Docker](#install-docker)
3. [Setup Data Directory](#setup-data-directory)
4. [Configure QRL Node (Optional)](#configure-qrl-node-optional)
5. [Pull Docker Image](#pull-docker-image)
6. [Run the Container](#run-the-container)
7. [Verify Installation](#verify-installation)
8. [Optional: Enable Public API Access](#optional-enable-public-api-access)
9. [Management Commands](#management-commands)
10. [Auto-Start with Systemd](#auto-start-with-systemd)

---

## Prerequisites

- Ubuntu server or VPS (tested on Ubuntu 24.04, should work on earlier versions)
- Root or sudo access
- At least 20GB free disk space (blockchain data grows over time)
- Stable internet connection
- Basic familiarity with Linux command line

---

## Install Docker

**Official Installation Guide:** https://docs.docker.com/engine/install/

**Example for Ubuntu:**

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

# Install Docker Engine
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify Docker installation
sudo docker run hello-world
```

---

## Setup Data Directory

Create a directory on your host system to store blockchain data persistently:

```bash
# Create data directory
sudo mkdir -p /opt/qrl-data

# The QRL node runs as user ID 999 inside the container
# Set ownership to match
sudo chown -R 999:999 /opt/qrl-data
```

---

## Configure QRL Node (Optional)

The QRL node will use default settings if no configuration file is provided. However, if you want to customize the node's behavior, it's recommended to download the official configuration template:

```bash
# Download example config
sudo curl -o /opt/qrl-data/config.yml https://docs.theqrl.org/assets/files/config-45439dd75cab079edd9e7cbd8a383183.yml

# Set proper ownership
sudo chown 999:999 /opt/qrl-data/config.yml
```

You can then uncomment and modify any settings you need. For available configuration options and their descriptions, refer to the official documentation: https://docs.theqrl.org/use/node/config

---

## Pull Docker Image

Download the official QRL Docker image from Docker Hub:

```bash
sudo docker pull qrledger/qrl-docker:jammy
```

---

## Run the Container

Create and start the QRL node container:

```bash
sudo docker run -d \
  --name=qrl-node \
  -v /opt/qrl-data:/home/qrl/.qrl \
  -p 19009:19009 \
  -p 19000:19000 \
  qrledger/qrl-docker:jammy \
  -l INFO
```

**Command explanation:**
- `-d` - Run in detached mode (background)
- `--name=qrl-node` - Name the container "qrl-node"
- `-v /opt/qrl-data:/home/qrl/.qrl` - Mount host directory to container for persistent data
- `-p 19009:19009` - Map API port (access controlled by QRL config inside container)
- `-p 19000:19000` - Map P2P port for blockchain networking
- `qrledger/qrl-docker:jammy` - The Docker image to use
- `-l INFO` - This overrides the default `--debug` flag which is not intended for production use, and sets logging to INFO level

Verify the container is running:

```bash
sudo docker ps
```

---

## Verify Installation

Check that your QRL node is running and syncing:

```bash
# View container logs
sudo docker logs -f qrl-node
# Press Ctrl+C to stop following

# Check node status
sudo docker exec -it qrl-node qrl state
```

**What to look for:**
- `state: SYNCING` or `state: SYNCED` - Node is working correctly
- `blockHeight` - Should be increasing as blockchain syncs
- `numConnections` - Should show connected peers

Initial blockchain sync can take several hours to days depending on network speed and current blockchain size.

---

## Optional: Enable Public API Access

By default, the QRL node's API only listens on localhost (127.0.0.1) inside the container for security. If you need to access the API from external applications, you can enable public access.

**⚠️ Security Note:** Enabling public API access allows anyone who can reach your server to query your node. This is generally safe for read-only operations but ensure you understand the implications.

### Enable Public API

1. Download the example configuration file if you haven't already (see [Configure QRL Node](#configure-qrl-node-optional) section above)

2. Edit the configuration file:

```bash
sudo nano /opt/qrl-data/config.yml
```

3. Find the line:

```yaml
# public_api_host: "127.0.0.1"
```

4. Uncomment it and change to:

```yaml
public_api_host: "0.0.0.0"
```

5. Save the file and restart the container:

```bash
sudo docker restart qrl-node
```

The API will now be accessible from external networks on port 19009.

---

## Management Commands

### Container Operations

```bash
# Start the container
sudo docker start qrl-node

# Stop the container
sudo docker stop qrl-node

# Restart the container
sudo docker restart qrl-node

# View container status
sudo docker ps

# Remove container (data in /opt/qrl-data remains safe)
sudo docker stop qrl-node
sudo docker rm qrl-node
```

### Logs and Monitoring

```bash
# View logs in real-time
sudo docker logs -f qrl-node

# View last 50 lines
sudo docker logs --tail 50 qrl-node

# Check node status
sudo docker exec -it qrl-node qrl state
```

### Updating

```bash
# Pull latest image
sudo docker pull qrledger/qrl-docker:jammy

# Stop and remove old container
sudo docker stop qrl-node
sudo docker rm qrl-node

# Start new container with same data
sudo docker run -d \
  --name=qrl-node \
  -v /opt/qrl-data:/home/qrl/.qrl \
  -p 19009:19009 \
  -p 19000:19000 \
  qrledger/qrl-docker:jammy \
  -l INFO
```

---

## Auto-Start with Systemd

To automatically start the QRL node on system boot:

```bash
sudo nano /etc/systemd/system/qrl-node.service
```

Paste this content:

```ini
[Unit]
Description=QRL Node Docker Container
Requires=docker.service
After=docker.service

[Service]
Type=simple
Restart=on-failure
RestartSec=5s
StartLimitIntervalSec=60
StartLimitBurst=3
KillMode=none

ExecStartPre=-/usr/bin/docker rm -f qrl-node
ExecStart=/usr/bin/docker run \
  --name=qrl-node \
  -v /opt/qrl-data:/home/qrl/.qrl \
  -p 19009:19009 \
  -p 19000:19000 \
  qrledger/qrl-docker:jammy \
  -l INFO

ExecStop=/usr/bin/docker stop qrl-node
ExecStopPost=-/usr/bin/docker rm -f qrl-node

[Install]
WantedBy=multi-user.target
```

Save and enable:

```bash
# Reload systemd
sudo systemctl daemon-reload

# Enable service to start on boot
sudo systemctl enable qrl-node.service

# Start service now
sudo systemctl start qrl-node.service

# Check status
sudo systemctl status qrl-node.service

# View logs
sudo journalctl -u qrl-node.service -f
```

---

## Additional Resources

- **QRL Documentation:** https://docs.theqrl.org/
- **QRL Node Configuration:** https://docs.theqrl.org/use/node/config
- **QRL Public API:** https://docs.theqrl.org/api/qrl-public-api/
- **QRL Explorer:** https://explorer.theqrl.org/
- **Docker Hub (QRL Image):** https://hub.docker.com/r/qrledger/qrl-docker
