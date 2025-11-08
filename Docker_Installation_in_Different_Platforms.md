# Docker Installation Guide - Ubuntu & macOS

A comprehensive step-by-step guide to installing Docker on Ubuntu and macOS systems.

---

## Table of Contents
1. [Ubuntu Installation](#ubuntu-installation)
2. [macOS Installation](#macos-installation)
3. [Windows Installation](#windows-installation)
4. [Amazon Linux Installation](#amazon-linux-installation)
5. [Post-Installation Steps](#post-installation-steps)
6. [Verification & Testing](#verification--testing)
7. [Troubleshooting](#troubleshooting)
8. [Uninstallation](#uninstallation)

---

# UBUNTU INSTALLATION

## Prerequisites

### System Requirements
- **OS:** Ubuntu 22.04 LTS (Jammy), 20.04 LTS (Focal), or 18.04 LTS (Bionic)
- **Architecture:** 64-bit (x86_64/amd64, arm64, armhf, s390x)
- **Kernel:** Minimum version 3.10
- **Storage Driver:** overlay2 (recommended)

### Check Your Ubuntu Version
```bash
lsb_release -a
```

### Check System Architecture
```bash
uname -m
```

### Check Kernel Version
```bash
uname -r
```

---

## Method 1: Install Using the Official Repository (Recommended)

### Step 1: Update Package Index
```bash
sudo apt-get update
```

### Step 2: Install Required Dependencies
```bash
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

**Explanation:**
- `ca-certificates` - Allows apt to use repositories over HTTPS
- `curl` - Tool to download files from the internet
- `gnupg` - GNU Privacy Guard for secure key management
- `lsb-release` - Provides Linux Standard Base version information

### Step 3: Add Docker's Official GPG Key
```bash
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

**Explanation:**
- Creates a directory for storing GPG keys
- Downloads Docker's official GPG key for package verification
- Converts the key to a format apt can use

### Step 4: Set Up the Docker Repository
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**Explanation:**
- Adds Docker's official repository to your system's software sources
- Automatically detects your Ubuntu version and architecture
- Uses the GPG key for package verification

### Step 5: Update Package Index Again
```bash
sudo apt-get update
```

### Step 6: Install Docker Engine, CLI, and Containerd
```bash
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Package Explanation:**
- `docker-ce` - Docker Community Edition (the main Docker engine)
- `docker-ce-cli` - Command-line interface for Docker
- `containerd.io` - Container runtime
- `docker-buildx-plugin` - BuildKit-based builder (multi-platform support)
- `docker-compose-plugin` - Docker Compose V2 plugin

**Installation Time:** Approximately 2-5 minutes depending on internet speed.

### Step 7: Verify Docker Engine Installation
```bash
sudo docker --version
```

**Expected Output:**
```
Docker version 24.0.x, build xxxxxxx
```

### Step 8: Check Docker Service Status
```bash
sudo systemctl status docker
```

**Expected Output:**
```
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since...
```

---

## Method 2: Install Using Convenience Script (Quick Method)

⚠️ **Warning:** This method is not recommended for production environments. Use only for testing or development.

### Step 1: Download and Run Installation Script
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

### Step 2: Verify Installation
```bash
docker --version
```

### Step 3: Clean Up Script
```bash
rm get-docker.sh
```

---

## Method 3: Install Specific Version

### Step 1: List Available Versions
```bash
apt-cache madison docker-ce
```

**Sample Output:**
```
docker-ce | 5:24.0.0-1~ubuntu.22.04~jammy | https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
docker-ce | 5:23.0.6-1~ubuntu.22.04~jammy | https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
```

### Step 2: Install Specific Version
```bash
# Replace VERSION_STRING with desired version (e.g., 5:24.0.0-1~ubuntu.22.04~jammy)
sudo apt-get install -y \
    docker-ce=VERSION_STRING \
    docker-ce-cli=VERSION_STRING \
    containerd.io \
    docker-buildx-plugin \
    docker-compose-plugin
```

### Step 3: Hold Package to Prevent Auto-Updates (Optional)
```bash
sudo apt-mark hold docker-ce docker-ce-cli
```

---

## Post-Installation Configuration (Ubuntu)

### Step 1: Enable Docker Service to Start on Boot
```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

### Step 2: Add Your User to the Docker Group
```bash
sudo usermod -aG docker $USER
```

**Explanation:**
- Adds your user to the `docker` group
- Allows running Docker commands without `sudo`
- Requires logout/login or restart to take effect

### Step 3: Apply Group Changes
```bash
newgrp docker
```

**Or logout and login again:**
```bash
# Logout
exit
# Login again
```

### Step 4: Configure Docker Daemon (Optional)
Create or edit `/etc/docker/daemon.json`:

```bash
sudo nano /etc/docker/daemon.json
```

**Recommended Configuration:**
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "default-address-pools": [
    {
      "base": "172.17.0.0/16",
      "size": 24
    }
  ],
  "storage-driver": "overlay2",
  "features": {
    "buildkit": true
  }
}
```

**Configuration Explanation:**
- `log-driver`: Sets logging mechanism
- `log-opts`: Limits log file size and rotation
- `default-address-pools`: Default IP ranges for containers
- `storage-driver`: Filesystem layer for images
- `buildkit`: Enables BuildKit for faster builds

### Step 5: Restart Docker Service
```bash
sudo systemctl restart docker
```

### Step 6: Configure Docker to Use Proxy (If Required)
Create systemd drop-in directory:
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
```

Create proxy configuration file:
```bash
sudo nano /etc/systemd/system/docker.service.d/http-proxy.conf
```

Add proxy settings:
```ini
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80"
Environment="HTTPS_PROXY=https://proxy.example.com:443"
Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.example.com"
```

Reload and restart:
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

---

# macOS INSTALLATION

## Prerequisites

### System Requirements
- **macOS Version:** macOS 11 (Big Sur) or newer
- **Hardware:** 
  - 2010 or newer Mac model
  - Intel processor or Apple Silicon (M1/M2/M3)
  - Minimum 4GB RAM (8GB recommended)
  - VT-x/AMD-v virtualization enabled in BIOS

### Check macOS Version
```bash
sw_vers
```

### Check Processor Type
```bash
uname -m
```
- `x86_64` = Intel processor
- `arm64` = Apple Silicon

---

## Method 1: Install Docker Desktop (Recommended)

Docker Desktop is the official Docker application for macOS, providing a complete development environment.

### Step 1: Download Docker Desktop

**For Intel Macs:**
1. Visit: https://www.docker.com/products/docker-desktop
2. Click "Download for Mac (Intel Chip)"
3. Or use direct link: https://desktop.docker.com/mac/main/amd64/Docker.dmg

**For Apple Silicon Macs (M1/M2/M3):**
1. Visit: https://www.docker.com/products/docker-desktop
2. Click "Download for Mac (Apple Silicon)"
3. Or use direct link: https://desktop.docker.com/mac/main/arm64/Docker.dmg

**Using Terminal (wget or curl):**
```bash
# For Intel Macs
curl -o Docker.dmg https://desktop.docker.com/mac/main/amd64/Docker.dmg

# For Apple Silicon
curl -o Docker.dmg https://desktop.docker.com/mac/main/arm64/Docker.dmg
```

### Step 2: Install Docker Desktop

**GUI Installation:**
1. Open the downloaded `Docker.dmg` file
2. Drag the Docker icon to the Applications folder
3. Open Applications folder
4. Double-click Docker.app to launch

**Terminal Installation:**
```bash
# Mount the DMG
sudo hdiutil attach Docker.dmg

# Copy to Applications
sudo cp -R /Volumes/Docker/Docker.app /Applications/

# Unmount the DMG
sudo hdiutil detach /Volumes/Docker

# Clean up
rm Docker.dmg
```

### Step 3: Launch Docker Desktop

**From Applications:**
- Open Finder → Applications → Docker.app
- Double-click to launch

**From Terminal:**
```bash
open /Applications/Docker.app
```

### Step 4: Accept License Agreement
- Read and accept the Docker Subscription Service Agreement
- Click "Accept" to continue

### Step 5: Grant Privileged Access
- Docker Desktop requires privileged access to install networking components
- Enter your macOS password when prompted
- Click "Install Helper"

### Step 6: Complete Setup
- Wait for Docker Engine to start (usually 30-60 seconds)
- Look for the Docker whale icon in the menu bar
- When active, it will show "Docker Desktop is running"

### Step 7: Sign In (Optional)
- Click the Docker whale icon in the menu bar
- Select "Sign in"
- Use your Docker Hub credentials or create a new account
- Benefits: Access to Docker Hub, sync preferences, additional features

---

## Method 2: Install Using Homebrew

Homebrew provides an alternative installation method for Docker Desktop.

### Step 1: Install Homebrew (If Not Already Installed)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### Step 2: Update Homebrew
```bash
brew update
```

### Step 3: Install Docker Desktop via Homebrew Cask
```bash
brew install --cask docker
```

**Installation Time:** 5-10 minutes depending on internet speed.

### Step 4: Launch Docker Desktop
```bash
open /Applications/Docker.app
```

### Step 5: Grant Permissions
- Enter password when prompted for privileged access
- Complete the setup wizard

---

## Method 3: Install Docker CLI Only (Lightweight)

For users who only need Docker CLI without Docker Desktop GUI.

### Step 1: Install Docker CLI via Homebrew
```bash
brew install docker
```

### Step 2: Install Docker Compose
```bash
brew install docker-compose
```

### Step 3: Install Docker Machine (For VM Management)
```bash
brew install docker-machine
```

### Step 4: Install VirtualBox (Required for Docker Machine)
```bash
brew install --cask virtualbox
```

### Step 5: Create Docker Machine
```bash
docker-machine create --driver virtualbox default
```

### Step 6: Configure Shell Environment
```bash
eval $(docker-machine env default)
```

### Step 7: Add to Shell Profile for Persistence
```bash
echo 'eval $(docker-machine env default)' >> ~/.zshrc
# Or for bash:
echo 'eval $(docker-machine env default)' >> ~/.bash_profile
```

---

## Post-Installation Configuration (macOS)

### Step 1: Verify Docker Desktop is Running
Check the menu bar for the Docker whale icon. It should show:
- Green icon = Running
- Animated icon = Starting
- Yellow icon = Issue detected

### Step 2: Access Docker Desktop Preferences
Click Docker whale icon → Preferences/Settings

**Key Settings to Configure:**

#### General Tab
- ✅ Start Docker Desktop when you log in
- ✅ Include VM in Time Machine backups (optional)
- ✅ Use Docker Compose V2

#### Resources Tab

**Advanced:**
- **CPUs:** Allocate based on your system (default: 2, recommended: 4-6)
- **Memory:** Allocate RAM (default: 2GB, recommended: 4-8GB)
- **Swap:** Set swap size (recommended: 1GB)
- **Disk image size:** Set maximum size (default: 60GB)

```
Recommended Settings for Development:
- CPUs: Half of your total cores
- Memory: 25-50% of total RAM
- For 16GB RAM Mac: 4-8GB for Docker
- For 32GB RAM Mac: 8-16GB for Docker
```

**File Sharing:**
- Add directories that need to be accessible to containers
- Default: `/Users`, `/tmp`, `/private`

#### Docker Engine Tab
Edit `daemon.json` configuration:

```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "features": {
    "buildkit": true
  },
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

### Step 3: Configure Docker CLI
The Docker CLI is automatically configured with Docker Desktop. Verify:

```bash
which docker
```

**Expected Output:**
```
/usr/local/bin/docker
```

### Step 4: Enable Kubernetes (Optional)
Docker Desktop includes Kubernetes:

1. Open Docker Desktop Preferences
2. Go to Kubernetes tab
3. Check "Enable Kubernetes"
4. Click "Apply & Restart"
5. Wait for Kubernetes to start (first time: 5-10 minutes)

### Step 5: Configure Docker for VS Code (Optional)
If using Visual Studio Code:

```bash
# Install Docker extension
code --install-extension ms-azuretools.vscode-docker
```

### Step 6: Set Up Shell Completion (Optional)

**For Zsh (default on macOS Catalina+):**
```bash
# Add to ~/.zshrc
if [ -f /Applications/Docker.app/Contents/Resources/etc/docker.zsh-completion ]; then
  autoload -Uz compinit
  compinit
  source /Applications/Docker.app/Contents/Resources/etc/docker.zsh-completion
fi
```

**For Bash:**
```bash
# Add to ~/.bash_profile
if [ -f /Applications/Docker.app/Contents/Resources/etc/docker.bash-completion ]; then
  source /Applications/Docker.app/Contents/Resources/etc/docker.bash-completion
fi
```

Apply changes:
```bash
source ~/.zshrc
# or
source ~/.bash_profile
```

---

# POST-INSTALLATION STEPS (Both Systems)

## Verify Installation

### Step 1: Check Docker Version
```bash
docker --version
```

**Expected Output:**
```
Docker version 24.0.x, build xxxxxxx
```

### Step 2: Check Docker Compose Version
```bash
docker compose version
```

**Expected Output:**
```
Docker Compose version v2.x.x
```

### Step 3: Display System-Wide Information
```bash
docker info
```

**Expected Output:**
```
Client:
 Context:    default
 Debug Mode: false

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 24.0.x
 Storage Driver: overlay2
 ...
```

### Step 4: Run Test Container
```bash
docker run hello-world
```

**Expected Output:**
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
...
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### Step 5: Run Interactive Ubuntu Container
```bash
docker run -it ubuntu bash
```

**Inside container:**
```bash
# Check you're inside the container
cat /etc/os-release
# Exit container
exit
```

### Step 6: Test Network Connectivity
```bash
docker run --rm alpine ping -c 3 google.com
```

### Step 7: Test Volume Mounting
```bash
# Create a test directory
mkdir -p ~/docker-test
echo "Hello Docker" > ~/docker-test/test.txt

# Mount and read file
docker run --rm -v ~/docker-test:/data alpine cat /data/test.txt
```

**Expected Output:**
```
Hello Docker
```

### Step 8: Test Port Mapping
```bash
# Run nginx web server
docker run -d -p 8080:80 --name test-nginx nginx

# Test in browser or curl
curl http://localhost:8080

# Clean up
docker stop test-nginx
docker rm test-nginx
```

---

## Configure Docker for Optimal Performance

### Ubuntu Optimization

#### 1. Enable Live Restore
Allows containers to keep running during Docker daemon updates:

```bash
sudo nano /etc/docker/daemon.json
```

Add:
```json
{
  "live-restore": true
}
```

#### 2. Configure Storage Driver
```bash
docker info | grep "Storage Driver"
```

If not using overlay2, configure it:
```json
{
  "storage-driver": "overlay2"
}
```

#### 3. Set Container Log Rotation
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

#### 4. Configure Default Bridge Network
```json
{
  "bip": "172.26.0.1/16",
  "default-address-pools": [
    {
      "base": "172.17.0.0/16",
      "size": 24
    }
  ]
}
```

#### 5. Enable User Namespace Remapping (Security)
```bash
sudo nano /etc/docker/daemon.json
```

Add:
```json
{
  "userns-remap": "default"
}
```

Restart Docker:
```bash
sudo systemctl restart docker
```

### macOS Optimization

#### 1. Optimize Resource Allocation
Docker Desktop → Preferences → Resources

**Recommended for 16GB RAM Mac:**
- CPUs: 4
- Memory: 6GB
- Swap: 1GB
- Disk: 60GB

**Recommended for 32GB RAM Mac:**
- CPUs: 8
- Memory: 12GB
- Swap: 2GB
- Disk: 100GB

#### 2. Enable File Sharing for Specific Directories Only
Docker Desktop → Preferences → Resources → File Sharing

Remove unnecessary directories to improve performance.

#### 3. Use gRPC FUSE for File Sharing (Apple Silicon)
Docker Desktop → Preferences → General
- ✅ Enable VirtioFS (for better file sharing performance)

#### 4. Enable Resource Saver Mode
Docker Desktop → Preferences → General
- ✅ Resource saver (pauses Docker Desktop when no containers running)

---

## Security Best Practices

### Ubuntu Security

#### 1. Configure AppArmor Profile
```bash
sudo aa-status | grep docker
```

#### 2. Enable SELinux (If Available)
```bash
# Check SELinux status
getenforce

# Set to enforcing
sudo setenforce 1
```

#### 3. Configure Firewall
```bash
# Allow Docker through UFW
sudo ufw allow 2375/tcp
sudo ufw allow 2376/tcp

# Reload firewall
sudo ufw reload
```

#### 4. Restrict Docker Socket Permissions
```bash
sudo chmod 660 /var/run/docker.sock
sudo chown root:docker /var/run/docker.sock
```

### macOS Security

#### 1. Keep Docker Desktop Updated
Docker Desktop → Check for Updates

#### 2. Review Privacy Settings
System Preferences → Security & Privacy → Files and Folders
- Ensure Docker has necessary permissions only

#### 3. Enable Firewall
System Preferences → Security & Privacy → Firewall
- Turn on firewall
- Click "Firewall Options"
- Unblock Docker when prompted

---

# VERIFICATION & TESTING

## Comprehensive Test Suite

### Test 1: Basic Container Operations
```bash
# Pull an image
docker pull alpine:latest

# List images
docker images

# Run container
docker run -d --name test-alpine alpine sleep 1000

# List running containers
docker ps

# Execute command in container
docker exec test-alpine echo "Hello from container"

# Stop container
docker stop test-alpine

# Remove container
docker rm test-alpine

# Remove image
docker rmi alpine:latest
```

### Test 2: Network Testing
```bash
# Create custom network
docker network create test-network

# Run two containers on same network
docker run -d --name web --network test-network nginx
docker run -d --name client --network test-network alpine sleep 1000

# Test connectivity
docker exec client ping -c 3 web

# Cleanup
docker stop web client
docker rm web client
docker network rm test-network
```

### Test 3: Volume Testing
```bash
# Create volume
docker volume create test-volume

# Run container with volume
docker run -d --name vol-test -v test-volume:/data alpine sleep 1000

# Write data to volume
docker exec vol-test sh -c "echo 'Test data' > /data/test.txt"

# Read data from volume
docker exec vol-test cat /data/test.txt

# Cleanup
docker stop vol-test
docker rm vol-test
docker volume rm test-volume
```

### Test 4: Docker Compose Testing
Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html

  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

Run test:
```bash
# Create html directory
mkdir html
echo "<h1>Docker Compose Test</h1>" > html/index.html

# Start services
docker compose up -d

# Check services
docker compose ps

# View logs
docker compose logs

# Test web server
curl http://localhost:8080

# Stop services
docker compose down

# Cleanup
rm -rf html
```

### Test 5: Multi-Stage Build Test
Create `Dockerfile`:

```dockerfile
# Build stage
FROM node:16 AS builder
WORKDIR /app
RUN echo '{"name": "test", "version": "1.0.0"}' > package.json
RUN echo 'console.log("Hello from Docker build test");' > index.js
RUN npm install

# Production stage
FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app .
CMD ["node", "index.js"]
```

Run test:
```bash
# Build image
docker build -t test-multistage .

# Run container
docker run --rm test-multistage

# Cleanup
docker rmi test-multistage
rm Dockerfile
```

---

# TROUBLESHOOTING

## Ubuntu Troubleshooting

### Issue 1: Docker Daemon Not Starting

**Symptoms:**
```bash
Cannot connect to the Docker daemon at unix:///var/run/docker.sock
```

**Solutions:**

**Check if Docker service is running:**
```bash
sudo systemctl status docker
```

**Start Docker service:**
```bash
sudo systemctl start docker
```

**Enable auto-start on boot:**
```bash
sudo systemctl enable docker
```

**Check for errors in logs:**
```bash
sudo journalctl -xeu docker.service
```

**Reset Docker service:**
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### Issue 2: Permission Denied

**Symptoms:**
```bash
Got permission denied while trying to connect to the Docker daemon socket
```

**Solutions:**

**Add user to docker group:**
```bash
sudo usermod -aG docker $USER
newgrp docker
```

**Or logout and login again**

**Verify group membership:**
```bash
groups $USER
```

### Issue 3: Storage Space Issues

**Check disk usage:**
```bash
docker system df
```

**Clean up:**
```bash
# Remove unused containers
docker container prune

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Complete cleanup
docker system prune -a --volumes
```

### Issue 4: Network Conflicts

**Symptoms:**
```bash
Error response from daemon: could not find an available, non-overlapping IPv4 address pool
```

**Solutions:**

**Check existing networks:**
```bash
docker network ls
docker network inspect bridge
```

**Modify default bridge network:**
```bash
sudo nano /etc/docker/daemon.json
```

Add:
```json
{
  "bip": "192.168.1.1/24",
  "default-address-pools": [
    {
      "base": "192.168.0.0/16",
      "size": 24
    }
  ]
}
```

**Restart Docker:**
```bash
sudo systemctl restart docker
```

### Issue 5: Container Can't Access Internet

**Check Docker network:**
```bash
docker network inspect bridge
```

**Check IP forwarding:**
```bash
cat /proc/sys/net/ipv4/ip_forward
```

**Enable IP forwarding:**
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

**Make permanent:**
```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

**Check iptables:**
```bash
sudo iptables -L -n | grep DOCKER
```

**Restart Docker:**
```bash
sudo systemctl restart docker
```

### Issue 6: Old Docker Version Installed

**Remove old versions:**
```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

**Follow installation steps from beginning**

---

## macOS Troubleshooting

### Issue 1: Docker Desktop Won't Start

**Symptoms:**
- Docker whale icon stuck in "Starting" state
- "Docker Desktop is starting..." message indefinitely

**Solutions:**

**Restart Docker Desktop:**
```bash
# Quit Docker Desktop
osascript -e 'quit app "Docker"'

# Wait a few seconds
sleep 5

# Start Docker Desktop
open /Applications/Docker.app
```

**Reset Docker Desktop:**
1. Click Docker whale icon
2. Select "Troubleshoot"
3. Click "Reset to factory defaults"
4. Confirm and wait for reset to complete

**Check for conflicting VPNs:**
- Some VPNs interfere with Docker networking
- Disconnect VPN and try starting Docker

**Check system logs:**
```bash
# View Docker logs
tail -f ~/Library/Containers/com.docker.docker/Data/log/vm/console.log

# Check for errors
grep -i error ~/Library/Containers/com.docker.docker/Data/log/vm/*.log
```

**Reinstall Docker Desktop:**
```bash
# Completely remove Docker Desktop
rm -rf ~/Library/Group\ Containers/group.com.docker
rm -rf ~/Library/Containers/com.docker.docker
rm -rf ~/.docker

# Reinstall from DMG
```

### Issue 2: High CPU Usage

**Check resource usage:**
```bash
docker stats
```

**Solutions:**

**Limit resources in Docker Desktop:**
- Open Docker Desktop Preferences
- Go to Resources → Advanced
- Reduce CPUs and Memory allocation

**Stop unnecessary containers:**
```bash
docker stop $(docker ps -q)
```

**Disable Kubernetes (if not needed):**
- Docker Desktop → Preferences → Kubernetes
- Uncheck "Enable Kubernetes"

**Check for runaway containers:**
```bash
docker stats --no-stream
```

### Issue 3: Disk Space Issues on macOS

**Check disk usage:**
```bash
docker system df
```

**View Docker Desktop disk image:**
```bash
ls -lh ~/Library/Containers/com.docker.docker/Data/vms/0/data/Docker.raw
```

**Clean up:**
```bash
docker system prune -a --volumes
```

**Increase disk image size:**
1. Docker Desktop → Preferences → Resources
2. Increase "Disk image size"
3. Click "Apply & Restart"

**Manually reclaim space:**
```bash
# Stop Docker Desktop first
docker run --privileged --pid=host docker/desktop-reclaim-space
```

### Issue 4: Port Already in Use

**Symptoms:**
```bash
Error starting userland proxy: listen tcp 0.0.0.0:8080: bind: address already in use
```

**Find process using port:**
```bash
sudo lsof -i :8080
```

**Kill process:**
```bash
kill -9 <PID>
```

**Or use different port:**
```bash
docker run -p 8081:80 nginx
```

### Issue 5: File Sharing Issues

**Symptoms:**
- Slow volume mounts
- Permission denied errors in containers

**Solutions:**

**Check file sharing settings:**
1. Docker Desktop → Preferences → Resources → File Sharing
2. Ensure your project directory is listed
3. Add if necessary

**Use cached or delegated mounts:**
```bash
# For better performance on macOS
docker run -v $(pwd):/app:cached myimage
```

**Reset file sharing:**
1. Docker Desktop → Troubleshoot
2. Click "Clean / Purge data"
3. Confirm and restart

### Issue 6: Apple Silicon (M1/M2/M3) Compatibility

**Symptoms:**
- Some images don't work
- "exec format error"

**Solutions:**

**Use platform flag:**
```bash
docker run --platform linux/amd64 <image>
```

**Build for multiple platforms:**
```bash
docker buildx build --platform linux/amd64,linux/arm64 -t myimage .
```

**Check image platform:**
```bash
docker inspect <image> | grep Architecture
```

**Pull correct architecture:**
```bash
docker pull --platform linux/arm64 nginx
```

---

## Common Issues (Both Systems)

### Issue: "Cannot remove container" Error

**Solution:**
```bash
# Force remove
docker rm -f <container_id>

# If still failing, stop first
docker stop <container_id>
docker rm <container_id>
```

### Issue: "No space left on device"

**Solution:**
```bash
# Check space
df -h

# Clean Docker
docker system prune -a --volumes

# Remove specific items
docker container prune
docker image prune -a
docker volume prune
docker network prune
```

### Issue: DNS Resolution Problems

**Solution:**

**Edit daemon.json:**
```json
{
  "dns": ["8.8.8.8", "8.8.4.4"]
}
```

**Test DNS:**
```bash
docker run --rm alpine nslookup google.com
```

### Issue: Slow Image Pulls

**Solutions:**

**Use Docker mirror
