Docker Offline Installition 
This script can install docker service and add current user to docker group
1: Copy Docker-installition.tar.gz
2: Create Script File with this contet :
```
#!/bin/bash

# A unified script for offline Docker installation and Charbot server deployment.

# Check for root privileges
if [ "$EUID" -ne 0 ]; then
  echo "This script must be run with sudo."
  exit 1
fi

# Determine the active user, which is important for group permissions.
ACTIVE_USER=${SUDO_USER:-$(whoami)}
echo "Detected active user: $ACTIVE_USER"

# Phase 1: Offline Docker Installation
echo ""
echo "--- Phase 1: Installing Docker from Offline Packages ---"
echo "Uncompressing Docker packages..."
tar -xzvf docker_installation.tar.gz
if [ $? -ne 0 ]; then
    echo "Error: Failed to uncompress 'docker_installation.tar.gz'. Please ensure the file is in the current directory."
    exit 1
fi

cd docker_installation_files || exit 1

echo "Installing Docker packages..."
sudo dpkg -i ./*.deb
sudo apt-get install -f # Fix any missing dependencies
if [ $? -ne 0 ]; then
    echo "Error: Docker installation failed. Please check for missing dependencies."
    exit 1
fi

echo "Docker installation completed successfully."
cd ..
```
