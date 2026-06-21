# Phase 2: Wazuh Server VM Setup

## Overview
This phase creates the **central hub** of your SOC lab. The Wazuh Server VM runs on Ubuntu 22.04 LTS and hosts three critical components:
- **Wazuh Manager** (analyzes logs and triggers alerts)
- **Wazuh Indexer** (stores data, powered by OpenSearch)
- **Wazuh Dashboard** (web UI for SOC operations)

All other VMs will send their logs here, making this the most important machine in your lab.

## Hardware Requirements for This VM
- **vCPU**: 4 cores
- **RAM**: 8 GB (minimum; 6 GB is the bare minimum but 8 GB is recommended)
- **Disk**: 100 GB (to store logs from agents, retention ~90 days)
- **OS**: Ubuntu 22.04 LTS
- **Network**: Host-Only (VMnet2 from Phase 1)
- **Static IP**: 192.168.100.10

---

## Step 1: Create the Ubuntu 22.04 VM

### 1.1 Start VM Creation
1. Open **VMware Workstation**
2. Click **File → New Virtual Machine**
3. Choose **Custom (advanced)** configuration
4. Click **Next**

### 1.2 Hardware Compatibility
- Select **Workstation X.X** (latest version shown)
- Click **Next**

### 1.3 Installation Media
- Select **Installer disc image file (ISO)**
- Click **Browse** and select your **Ubuntu 22.04 LTS ISO** file
  - Download from https://releases.ubuntu.com/22.04/ if you don't have it
- Click **Next**

### 1.4 Guest Operating System
- Select **Linux**
- Version: **Ubuntu 64-bit**
- Click **Next**

### 1.5 Virtual Machine Name and Location
- **Name**: `wazuh-server` (or similar)
- **Location**: Choose a folder with enough space (100 GB minimum)
- Click **Next**

### 1.6 Processor Configuration
- **Processors**: 4
- **Cores per processor**: 1 (total = 4 vCPU)
- Click **Next**

### 1.7 Memory Configuration
- **Memory**: 8192 MB (8 GB)
- Click **Next**

### 1.8 Network Type
- Select **Custom**
- Dropdown: Select **VMnet2** (the Host-Only network from Phase 1)
- Click **Next**

### 1.9 I/O Controller Types
- Keep defaults (SCSI)
- Click **Next**

### 1.10 Virtual Disk Type
- Select **SCSI** (recommended)
- Click **Next**

### 1.11 Disk Capacity
- **Size**: 100 GB
- Select **Store virtual disk as a single file**
- Click **Next**

### 1.12 Finish
- Review all settings
- Click **Finish**

The VM will be created and Ubuntu installer will boot automatically.

---

## Step 2: Install Ubuntu 22.04 LTS

### 2.1 Ubuntu Installer Welcome Screen
1. Select **English** language
2. Click **Install Ubuntu Server**

### 2.2 Keyboard Layout
- Select your keyboard layout (e.g., English US)
- Click **Done**

### 2.3 Network Connections
This is where you set the static IP. Look for your network interface:

1. Click on the network interface shown (usually `ens33` or `eth0`)
2. Select **Edit IPv4**
3. Change from **Automatic (DHCP)** to **Manual**
4. Configure:
   ```
   Subnet: 192.168.100.0/24
   Address: 192.168.100.10
   Gateway: 192.168.100.1
   Name servers: 8.8.8.8, 8.8.4.4
   ```
5. Click **Save**
6. Click **Done**

### 2.4 Proxy Configuration
- Leave blank
- Click **Done**

### 2.5 Archive Mirror
- Keep default Ubuntu mirror
- Click **Done**

### 2.6 Storage Configuration
- Select default disk configuration
- Click **Done**
- Review the summary (should show `/dev/sda 100GB`)
- Click **Done**
- Confirm destructive action (writing to disk)

### 2.7 Profile Setup
This creates the main user account:

```
Your name: wazuh
Your server's name: wazuh-server
Pick a username: wazuh
Choose a password: [create a strong password]
Confirm your password: [re-enter password]
```

Keep this password safe — you'll need it for sudo commands.

Click **Done**

### 2.8 SSH Setup
- Check **Install OpenSSH server** (recommended for remote access)
- Click **Done**

### 2.9 Featured Server Snaps
- Leave unchecked (no additional snaps needed)
- Click **Done**

### 2.10 Installation Complete
- Wait for installation to finish (5-10 minutes)
- Click **Reboot Now**

The VM will restart and boot into Ubuntu 22.04.

---

## Step 3: Post-Installation Setup

### 3.1 Login to Ubuntu
After reboot:
```
ubuntu login: wazuh
Password: [enter your password]
```

### 3.2 Update System Packages
```bash
sudo apt update
sudo apt upgrade -y
```

This may take a few minutes. After completion, you'll see the prompt again.

### 3.3 Verify Network Configuration
```bash
hostname -I
```

You should see: `192.168.100.10`

If you don't see this IP, your static IP configuration didn't apply. See troubleshooting section below.

### 3.4 Verify Internet Connectivity (Optional)
```bash
ping 8.8.8.8
```

Press `Ctrl+C` to stop. You should see successful responses.

---

## Step 4: Install Wazuh All-in-One

### 4.1 Download and Run Installation Assistant

Run the verified Wazuh installation command:

```bash
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
```

**What this does:**
- Downloads the Wazuh installation script (version 4.14)
- Runs it with the `-a` flag (all-in-one: manager, indexer, dashboard)
- Automatically detects your OS and installs appropriate packages
- Configures certificates for secure communication
- Starts all Wazuh services

**Time**: 15-25 minutes (depends on internet speed and VM performance)

**Output**: While running, you'll see:
```
INFO: Downloading Wazuh installation files...
INFO: Installing Wazuh manager...
INFO: Installing Wazuh indexer...
INFO: Installing Wazuh dashboard...
INFO: Starting services...
INFO: --- Summary ---
INFO: You can access the web interface https://<WAZUH_DASHBOARD_IP>
User: admin
Password: <ADMIN_PASSWORD>
INFO: Installation finished.
```

**Important**: Copy the password shown (you'll need it for the dashboard login).

### 4.2 Store Password in a Text File

Create a safe location for your credentials:

```bash
mkdir -p ~/wazuh-credentials
sudo tar -O -xvf wazuh-install-files.tar wazuh-install-files/wazuh-passwords.txt > ~/wazuh-credentials/passwords.txt
cat ~/wazuh-credentials/passwords.txt
```

This file contains all passwords. Example output:
```
'admin' : 'abc123XYZ'
'wazuh' : 'def456UVW'
'logstash' : 'ghi789RST'
```

**Security Note**: Keep this file safe. In production, you'd encrypt it.

### 4.3 Verify Wazuh Services are Running

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

Each should show: `Active (running)`

If any show inactive, check logs:
```bash
sudo tail -50 /var/ossec/logs/ossec.log
```

---

## Step 5: Access Wazuh Dashboard from Your Host PC

### 5.1 Find Your VM's IP (if different from 192.168.100.10)

Inside the VM, verify the IP:
```bash
hostname -I
```

### 5.2 Open Browser on Your Host PC

From your **host PC** (not the VM):
1. Open **Chrome, Firefox, or Edge**
2. Go to: `https://192.168.100.10`

**Note**: You'll see a certificate warning (because it's self-signed). This is normal for lab environments.
- Chrome: Click **Advanced → Proceed to 192.168.100.10 (unsafe)**
- Firefox: Click **Advanced → Accept the Risk and Continue**

### 5.3 Login to Dashboard

When the Wazuh dashboard appears:
- **Username**: `admin`
- **Password**: [the password from Step 4.2]

You should see the Wazuh dashboard home page with:
- Security Events section (empty for now)
- Agent status (0 agents connected)
- Overview charts

---

## Step 6: Verify Wazuh Installation

### 6.1 Check Manager Status via CLI

```bash
sudo /var/ossec/bin/wazuh-control status
```

Expected output:
```
wazuh-remoted is running
wazuh-analysisd is running
wazuh-syscheckd is running
wazuh-agentd is not running (this is normal on server)
wazuh-logcollector is running
wazuh-exec is running
wazuh-modulesd is running
wazuh-authd is running
```

### 6.2 Check Indexer Health (via CLI)

```bash
sudo curl -k -u admin:$(grep "'admin'" ~/wazuh-credentials/passwords.txt | awk '{print $NF}' | tr -d "'") https://localhost:9200/_cluster/health?pretty
```

You should get output showing:
```
"status" : "green"
"number_of_nodes" : 1
"active_shards" : 20
```

This means the Wazuh Indexer is healthy.

### 6.3 Test Agent Enrollment (Dry Run)

Test that the agent enrollment system is working:

```bash
sudo /var/ossec/bin/wazuh-control info
```

Should show manager version, build number, and status.

---

## Troubleshooting

### Issue: Static IP not applied (hostname -I shows different IP)
**Solution**:
1. Edit netplan config:
```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

2. Ensure it contains:
```yaml
network:
  version: 2
  ethernets:
    ens33:  # or your interface name
      dhcp4: no
      addresses:
        - 192.168.100.10/24
      gateway4: 192.168.100.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

3. Save (Ctrl+X, Y, Enter)
4. Apply changes:
```bash
sudo netplan apply
sudo systemctl restart networking
```

5. Verify:
```bash
hostname -I
```

### Issue: Dashboard not accessible from host PC
**Solution**:
1. Verify VM IP is correct: `hostname -I` inside VM
2. From host PC command line, try: `ping 192.168.100.10`
3. If ping fails, check VMware network settings (go back to Phase 1)
4. Check firewall on host (Windows/Mac firewall might block)

### Issue: Wazuh installation failed or services not running
**Solution**:
1. Check installation logs:
```bash
sudo tail -100 /var/ossec/logs/ossec.log
```

2. Check disk space (Wazuh needs ~10GB minimum free):
```bash
df -h
```

3. Check RAM usage:
```bash
free -h
```

4. If services crashed, restart them:
```bash
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-dashboard
```

### Issue: "Certificate verify failed" when accessing dashboard
**Solution**: This is normal. Self-signed certificates always trigger warnings.
- Accept the security warning in your browser
- Certificate is valid even though not signed by a trusted CA
- In production, you'd use valid certificates

---

## Performance Tuning (Optional)

If the VM seems slow:

### 1. Increase allocated RAM (in VMware):
1. Power off the VM
2. Right-click VM → Settings
3. Change Memory to 12 GB
4. Power on

### 2. Increase Wazuh Indexer JVM Heap (if needed):
```bash
sudo nano /etc/wazuh-indexer/jvm.options.d/heap-size.options
```

Look for:
```
-Xms4g
-Xmx4g
```

If you increased VM RAM, increase these to match (e.g., to 6g if you have 12GB VM).

Restart:
```bash
sudo systemctl restart wazuh-indexer
```

---

## What Happens Next

After completing this phase:
1. ✅ Wazuh Server is running and accessible
2. ✅ All three components (manager, indexer, dashboard) are operational
3. ✅ You can log into the dashboard
4. ✅ The server is ready to receive logs from agents

Next: **Phase 3: Windows 10 Victim VM Setup** — where you'll create the target machine and install the Wazuh agent on it.

---

## Verification Checklist

- [ ] VMware VM created with correct specs (4 vCPU, 8GB RAM, 100GB disk, VMnet2)
- [ ] Ubuntu 22.04 LTS installed successfully
- [ ] Static IP set to 192.168.100.10 (verified with `hostname -I`)
- [ ] System packages updated (`apt update` and `apt upgrade`)
- [ ] Wazuh installation script downloaded and executed
- [ ] Installation completed without errors
- [ ] Passwords saved to ~/wazuh-credentials/passwords.txt
- [ ] All three services running (`systemctl status` shows active for manager, indexer, dashboard)
- [ ] Dashboard accessible from host PC at https://192.168.100.10
- [ ] Can log in with admin credentials
- [ ] Dashboard shows "0 agents connected" (expected at this stage)
- [ ] Manager status shows all services running (`/var/ossec/bin/wazuh-control status`)
- [ ] Indexer health shows green status (curl check)
- [ ] No disk space issues (`df -h` shows >20GB free)
- [ ] Understand the Wazuh Server role in the lab architecture

---

## Reference Commands Quick List

```bash
# View Wazuh manager status
sudo /var/ossec/bin/wazuh-control status

# View Wazuh manager info
sudo /var/ossec/bin/wazuh-control info

# Check indexer health
sudo curl -k https://localhost:9200/_cluster/health?pretty

# View Wazuh logs
sudo tail -50 /var/ossec/logs/ossec.log

# Restart a service
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-dashboard

# View system resources
free -h
df -h
```

---

## Next Phase
Proceed to **Phase 3: Windows 10 Victim VM Setup** when all checklist items are complete.
