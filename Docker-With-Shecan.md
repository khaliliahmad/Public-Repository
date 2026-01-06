# Bash Script for DNS Configuration and Docker Installation

You can create a Bash script that performs the desired steps for configuring DNS settings and installing Docker. Below is an example of this script:

```bash
#!/bin/bash

echo "Step 1: Updating DNS settings in /etc/systemd/resolved.conf"
sudo bash -c 'echo -e "DNS=178.22.122.100 185.51.200.2\nFallbackDNS=8.8.8.8" > /etc/systemd/resolved.conf'
echo "DNS settings updated."

echo "Step 2: Restarting systemd-resolved service"
sudo systemctl restart systemd-resolved
echo "systemd-resolved service restarted."

echo "Step 3: Creating a symbolic link for resolv.conf"
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
echo "Symbolic link created."

echo "Step 4: Installing Docker"
curl -fsSL https://get.docker.com | sh
echo "Docker installation completed."

echo "Step 5: Restoring DNS settings to default"
sudo bash -c 'sed -i "s/^DNS=/\\#DNS=/; s/^FallbackDNS=/\\#FallbackDNS=/" /etc/systemd/resolved.conf'
echo "DNS settings restored to default."
```

## Important Notes
1. **Execution Permissions**: After creating this file, you will need to grant execution permissions to it. You can use the following command:
   ```bash
   chmod +x your_script.sh
   ```

2. **Execution**: To run the script, use the following command:
   ```bash
   ./your_script.sh
   ```

## Step Descriptions
- **Step 1**: Adds the DNS settings to the `resolved.conf` file.
- **Step 2**: Restarts the `systemd-resolved` service.
- **Step 3**: Creates a symbolic link for `resolv.conf`.
- **Step 4**: Installs Docker.
- **Step 5**: Restores the DNS settings to default and comments out the added lines.

This script will inform you about what is happening at each stage.

