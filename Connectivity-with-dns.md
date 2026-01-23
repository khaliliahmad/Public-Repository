# Network Connectivity Monitor 

**Author:** AhmadKhalili

**Date:** 2025-12-24

**Description:** A Bash script designed to monitor the reachability of specific server IPs. If a connection times out, it automatically runs diagnostics against a comprehensive list of Iranian and International DNS providers to pinpoint network issues.

---

## 1. The Script (`network_monitor.sh`)

Copy the code below into a file named `network_monitor.sh`.

```bash
#!/bin/bash

# ==========================================
# Network Monitor for @metacena
# Created: 2025-12-24
# ==========================================

# Colors for output
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# 1. Define Target IPs
TARGET_IPS=(
    "IP1"
    "IP2"
    "IP3"
    "IP4"
    "IP5"
    "IP6"
    "IP7"
    "IP8"
    "IP9"
    "IP10"
    "IP11"
    "IP12"
    "IP13"
    "IP14"
    "IP15"
)

# 2. Define DNS Servers
declare -A DNS_SERVERS

# Standard List
DNS_SERVERS["OpenDNS_1"]="208.67.222.222"
DNS_SERVERS["OpenDNS_2"]="208.67.220.220"
DNS_SERVERS["Shatel_1"]="85.15.1.14"
DNS_SERVERS["Shatel_2"]="85.15.1.15"
DNS_SERVERS["Mokhaberat_1"]="217.218.155.155"
DNS_SERVERS["Mokhaberat_2"]="217.218.127.127"
DNS_SERVERS["Mokhaberat_3"]="2.188.21.130"
DNS_SERVERS["Mokhaberat_4"]="2.188.21.131"
DNS_SERVERS["Mokhaberat_5"]="2.188.21.132"
DNS_SERVERS["Mokhaberat_AZ_1"]="217.219.72.194"
DNS_SERVERS["Asiatech/Shecan_1"]="178.22.122.100"
DNS_SERVERS["Asiatech/Shecan_2"]="185.51.200.2"
DNS_SERVERS["ArvanCloud"]="185.51.200.3"
DNS_SERVERS["DerakCloud_1"]="193.151.128.100"
DNS_SERVERS["DerakCloud_2"]="193.151.128.200"
DNS_SERVERS["Respina_1"]="10.202.10.202"
DNS_SERVERS["Respina_2"]="10.202.10.102"
DNS_SERVERS["Faradade"]="81.91.144.116"
DNS_SERVERS["Amirkabir_Uni"]="185.51.200.4"
DNS_SERVERS["NIC_ir_A"]="193.189.123.2"
DNS_SERVERS["NIC_ir_B"]="193.189.122.83"
DNS_SERVERS["NIC_ir_D"]="194.225.70.83"
DNS_SERVERS["DNS_Pro"]="87.107.110.110"
DNS_SERVERS["Electro"]="78.157.42.100"
DNS_SERVERS["Pishgaman"]="5.202.100.100"
DNS_SERVERS["asiatech1"]="185.98.113.113"
DNS_SERVERS["asiatech2"]="185.98.114.114"

# Adding Mokhaberat Azerbaijan range 133-139
for i in {133..139}; do 
    DNS_SERVERS["Mokhaberat_AZ_Range_$i"]="2.185.239.$i"
done

# ==========================================
# Start Execution
# ==========================================

echo -e "${BLUE}Starting Network Connectivity Test...${NC}"
echo "Timestamp: $(date)"
echo "---------------------------------------------"

for ip in "${TARGET_IPS[@]}"; do
    # Try to ping with a timeout of 2 seconds, sending 3 packets
    if ping -c 3 -W 2 "$ip" > /dev/null 2>&1; then
        echo -e "[ ${GREEN}OK${NC} ] $ip is reachable."
    else
        echo -e "[ ${RED}FAIL${NC} ] $ip TIMED OUT."
        echo -e "${YELLOW}    -> Investigating via DNS List...${NC}"
        
        # Iterate through all DNS servers to check connectivity
        printf "%-30s %-18s %-20s\n" "DNS Name" "DNS IP" "Reverse Lookup Status"
        echo "    ------------------------------------------------------------------"
        
        for dns_name in "${!DNS_SERVERS[@]}"; do
            dns_ip=${DNS_SERVERS[$dns_name]}
            
            # Use 'dig' to do a reverse lookup on the Target IP using the specific DNS server
            lookup_result=$(dig -x "$ip" @"$dns_ip" +short +time=1 +tries=1 2>/dev/null)
            
            if [ $? -eq 0 ] && [ ! -z "$lookup_result" ]; then
                # DNS is reachable AND resolved the IP
                printf "    %-30s %-18s ${GREEN}RESOLVED${NC} (%s)\n" "$dns_name" "$dns_ip" "$lookup_result"
            elif [ $? -eq 0 ]; then
                # DNS is reachable but returned no record
                printf "    %-30s %-18s ${BLUE}REACHABLE (No Record)${NC}\n" "$dns_name" "$dns_ip"
            else
                # DNS server itself is unreachable or timed out
                printf "    %-30s %-18s ${RED}DNS TIMEOUT${NC}\n" "$dns_name" "$dns_ip"
            fi
        done
        echo ""
    fi
done

echo "---------------------------------------------"
echo "Test Complete."

```

## 2. Installation & Usage

1. **Create the file:**
```bash
nano network_monitor.sh

```


*(Paste the code above into the terminal)*
2. **Make the script executable:**
```bash
chmod +x network_monitor.sh

```


3. **Run the monitor:**
```bash
./network_monitor.sh

```



## 3. How It Works

* **Step 1: Ping Test**
The script attempts to ping each IP in your target list (Exchange servers, VPS, etc.).
* **Step 2: Failure Handling**
If a Ping fails (Timeout), the script assumes the direct route is blocked or the server is down.
* **Step 3: DNS Cross-Check**
It immediately switches to "Investigation Mode." It tries to reach the failed IP via **Reverse DNS Lookup** using a variety of ISP DNS servers (Shatel, Mokhaberat, Shecan, Electro, etc.).
* **Resolved:** Means your internet is working and can reach that DNS, but the direct route to the specific IP is blocked.
* **DNS Timeout:** Means your internet connection itself might be unstable or cut off from that specific ISP network.
