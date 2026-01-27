
# Ubuntu DNS Optimizer for Fintech Environments

A comprehensive bash utility designed to diagnose, benchmark, and optimize DNS resolution in restricted network environments. This tool is specifically tuned for servers interacting with Iranian financial gateways (Paystar, DirectPay) and SMS infrastructure (IPPanel).

## 🚀 Overview

In many datacenter environments, standard DNS providers (like Google or OpenDNS) are either throttled, blocked, or suffer from high latency. This script automates the process of:
1.  **Benchmarking:** Testing a curated list of reliable Iranian and international DNS providers.
2.  **Selection:** Identifying the server with the lowest latency.
3.  **Automation:** Safely updating `systemd-resolved` configurations while preserving backups.
4.  **Bypassing Stubs:** Fixing the common `127.0.0.53` "Connection Refused" error by linking directly to the resolved configuration.

## 🛠️ The Script (`optimize_dns.sh`)

```bash
#!/bin/bash

# --- CONFIGURATION ---
TARGET="core.paystar.ir"
FILE="/etc/systemd/resolved.conf"
declare -A DNS_SERVERS

# Define Providers
DNS_SERVERS["Shecan_Primary"]="178.22.122.100"
DNS_SERVERS["Shecan_Secondary"]="185.51.200.2"
DNS_SERVERS["Mokhaberat_1"]="217.218.155.155"
DNS_SERVERS["Mokhaberat_2"]="217.218.127.127"
DNS_SERVERS["DNS_Pro"]="87.107.110.110"
DNS_SERVERS["Electro"]="78.157.42.100"
DNS_SERVERS["Pishgaman"]="5.202.100.100"

# Automatically add Mokhaberat Azerbaijan range
for i in {133..139}; do 
    DNS_SERVERS["Mokhaberat_AZ_$i"]="2.185.239.$i"
done

# --- PHASE 1: TESTING ---
echo "---------------------------------------------------------------------------------------"
echo "PHASE 1: Benchmarking DNS providers against $TARGET..."
printf "%-25s | %-16s | %-10s | %s\n" "Provider" "IP Address" "Status" "Latency"
echo "---------------------------------------------------------------------------------------"

BEST_DNS=""
BEST_TIME=999

for name in "${!DNS_SERVERS[@]}"; do
    ip=${DNS_SERVERS[$name]}
    
    START_TIME=$(date +%s%N)
    result=$(dig @$ip $TARGET +short +time=1 +tries=1 2>&1)
    END_TIME=$(date +%s%N)
    ELAPSED=$(( (END_TIME - START_TIME) / 1000000 ))

    if [[ $result != *";; communications error"* ]] && [[ $result != *";; connection timed out"* ]] && [[ -n "$result" ]]; then
        status="SUCCESS"
        printf "%-25s | %-16s | %-10s | %dms\n" "$name" "$ip" "$status" "$ELAPSED"
        
        if [ $ELAPSED -lt $BEST_TIME ]; then
            BEST_TIME=$ELAPSED
            BEST_DNS=$ip
        fi
    else
        printf "%-25s | %-16s | %-10s | ---\n" "$name" "$ip" "FAILED"
    fi
done

if [ -z "$BEST_DNS" ]; then
    echo "CRITICAL ERROR: No working DNS found. Aborting update."
    exit 1
fi

echo -e "\nWinner: $BEST_DNS (${BEST_TIME}ms)"
echo "---------------------------------------------------------------------------------------"

# --- PHASE 2: UPDATING SYSTEM ---
echo "PHASE 2: Updating System Configuration..."
echo "Backing up $FILE..."
sudo cp $FILE "${FILE}.bak_$(date +%H%M%S)"

echo "Commenting out legacy DNS lines..."
# Comment existing DNS/FallbackDNS lines to prevent conflicts
sudo sed -i 's/^\(DNS=.*\)/#\1/' $FILE
sudo sed -i 's/^\(FallbackDNS=.*\)/#\1/' $FILE

echo "Injecting optimal DNS settings..."
# Insert the new DNS line under [Resolve] section with redundancy
sudo sed -i "/\[Resolve\]/a DNS=$BEST_DNS 178.22.122.100 78.157.42.100" $FILE

echo "Restarting systemd-resolved and updating symlink..."
sudo systemctl restart systemd-resolved
# Create a direct link to bypass the local stub resolver
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf

echo -e "\n--- FINAL VERIFICATION ---"
resolvectl status | grep "DNS Servers" -A 1
echo "Testing connectivity to Fintech API..."
curl -I -m 5 [https://core.paystar.ir](https://core.paystar.ir) 2>&1 | grep "HTTP" || echo "Note: HTTP check failed, but DNS may be resolved."

```

## ⚙️ Installation & Usage

1. **Clone or Copy** the script to your server.
2. **Grant Execution Permissions**:
```bash
chmod +x optimize_dns.sh

```


3. **Execute with Sudo**:
```bash
sudo ./optimize_dns.sh

```



## 📝 Why this works

* **Remote DNS Resolution**: By using `Shecan` or `Electro`, you ensure that DNS queries for blocked domains are resolved correctly via their tunneling infrastructure.
* **Resolved.conf vs Netplan**: While Netplan is good for static IPs, editing `resolved.conf` directly provides more granular control over how the OS handles fallbacks and caching.
* **Symlink Strategy**: Pointing `/etc/resolv.conf` to `/run/systemd/resolve/resolv.conf` (instead of `stub-resolv.conf`) eliminates the local loopback delay and the common "Connection Refused" issues encountered in high-load Docker environments.
