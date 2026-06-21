# Phase 3: Windows 10 Victim VM Setup

## Overview
This phase creates the **victim machine** in your SOC lab. This Windows 10 VM will:
- Represent a typical endpoint in a corporate network
- Run the Wazuh Agent to send logs to the Wazuh Server
- Be the target of simulated attacks from your Kali attacker
- Generate security events that you'll detect and analyze

**Important**: You are NOT installing anything on the Wazuh Server itself. Only on this separate Windows 10 VM.

## Hardware Requirements for This VM
- **vCPU**: 2 cores
- **RAM**: 4 GB
- **Disk**: 60 GB
- **OS**: Windows 10 Pro or Home (x64)
- **Network**: Host-Only (VMnet2 from Phase 1)
- **Static IP**: 192.168.100.20

---

## Step 1: Create the Windows 10 VM

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
- Click **Browse** and select your **Windows 10 x64 ISO**
  - You need a Windows 10 installation ISO (Home, Pro, or Enterprise)
  - If you don't have one, download from Microsoft's official site
  - Windows 10 evaluation ISOs are available from: https://www.microsoft.com/en-us/software-download/windows10
- Click **Next**

### 1.4 Guest Operating System
- Select **Microsoft Windows**
- Version: **Windows 10 x64**
- Click **Next**

### 1.5 Virtual Machine Name and Location
- **Name**: `windows-victim` (or similar)
- **Location**: Choose a folder with enough space (60 GB minimum)
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
- **Size**: 60 GB
- Select **Store virtual disk as a single file**
- Click **Next**

### 1.12 Finish
- Review all settings
- Click **Finish**

The Windows 10 installer will boot automatically.

---

## Step 2: Install Windows 10

### 2.1 Windows Setup Wizard
1. Select your language (English)
2. Click **Next**
3. Click **Install now**

### 2.2 Accept License Terms
- Check **I accept the license terms**
- Click **Next**

### 2.3 Installation Type
- Select **Custom: Install Windows only (advanced)**
- Select the disk shown (should be one 60GB disk)
- Click **Next**

Windows will install. The VM will restart several times. Wait until you see the **Windows 10 Setup** screen (not the installer).

### 2.4 Region and Language
- Select your region and keyboard layout
- Click **Next**

### 2.5 Network Connection
- Select **I don't have internet** (we're on an isolated network)
- Click **Continue with limited setup**

### 2.6 Account Setup
- **Account name**: `wazuh` (or your preferred name)
- **Password**: [create a password]
- **Password hint**: (optional)
- Click **Next**

**Keep this password safe** — you'll need it to log into Windows.

### 2.7 Privacy Settings
- Disable all tracking options (for a clean lab environment):
  - Turn off advertising ID
  - Turn off tailored experiences
  - Turn off diagnostic data (set to "Required diagnostic data")
- Click **Accept**

### 2.8 Complete Setup
Windows will finish setup and show the desktop. Great! You now have Windows 10 running.

---

## Step 3: Network Configuration

### 3.1 Set Static IP Address

1. Right-click **Start menu** → **Settings**
2. Go to **Network & Internet**
3. Click **Change adapter options**
4. Right-click your network adapter (usually "Ethernet") → **Properties**
5. Select **Internet Protocol Version 4 (TCP/IPv4)**
6. Click **Properties**
7. Select **Use the following IP address**
8. Fill in:
   ```
   IP address: 192.168.100.20
   Subnet mask: 255.255.255.0
   Default gateway: 192.168.100.1
   ```
9. In **Preferred DNS server**, enter: `8.8.8.8`
10. In **Alternate DNS server**, enter: `8.8.4.4`
11. Click **OK** → **OK** → **OK**

### 3.2 Verify Network Configuration

Open **Command Prompt** (right-click Start → Command Prompt (Admin)):

```cmd
ipconfig /all
```

You should see:
```
IPv4 Address: 192.168.100.20
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.100.1
```

Verify connectivity to Wazuh Server:

```cmd
ping 192.168.100.10
```

You should see successful responses like:
```
Reply from 192.168.100.10: bytes=32 time=<1ms TTL=64
```

---

## Step 4: Prepare Windows for Log Collection

### 4.1 Enable Windows Event Logging

This ensures Windows generates rich security logs that Wazuh can collect.

1. Press **Windows key + R**
2. Type: `gpedit.msc`
3. Press **Enter**
4. Navigate to: **Computer Configuration → Windows Settings → Security Settings → Advanced Audit Policy Configuration → Audit Policies**
5. Enable the following policies by setting to **Success and Failure**:
   - Account Logon
   - Account Management
   - Process Creation
   - Logon/Logoff

**Alternative (if gpedit not available on Home edition)**:

Open **PowerShell as Admin** and run:

```powershell
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
auditpol /set /subcategory:"Account Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Logon/Logoff" /success:enable /failure:enable
```

### 4.2 Enable Windows Defender (Optional but Recommended)

1. **Settings → Update & Security → Windows Security**
2. Click **Virus & threat protection**
3. Verify **Real-time protection** is ON (shows green)

Wazuh will capture these alerts in real time.

### 4.3 Install Sysmon (Optional but Powerful)

Sysmon provides deep visibility into process execution and network connections. It's not required but highly recommended.

**Download Sysmon**:
1. From a web browser in Windows, go to: https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon
2. Download the ZIP file
3. Extract to a folder (e.g., C:\Tools\Sysmon)

**Install Sysmon**:
1. Open **PowerShell as Admin**
2. Navigate to the Sysmon folder:
```powershell
cd C:\Tools\Sysmon
```

3. Download a Sysmon configuration from SwiftOnSecurity (recommended):
```powershell
$url = "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml"
Invoke-WebRequest -Uri $url -OutFile sysmonconfig.xml
```

4. Install Sysmon with the config:
```powershell
.\sysmon64.exe -i -c sysmonconfig.xml
```

You should see: `System Monitor installed`

### 4.4 Disable Windows Firewall (Lab-Only, for simplicity)

Since this is an isolated lab, disabling the firewall simplifies troubleshooting:

1. **Settings → Update & Security → Windows Security**
2. Click **Firewall & network protection**
3. Click **Windows Defender Firewall**
4. **Turn off** for all networks (Domain, Private, Public)

**Note**: In production, you'd never do this. This is lab-specific.

---

## Step 5: Install Wazuh Agent

### 5.1 Download Wazuh Agent from Dashboard

The easiest way is to deploy the agent directly from the Wazuh dashboard:

1. From your **host PC**, go to: `https://192.168.100.10`
2. Log in with admin credentials
3. Click **Agents** (left sidebar)
4. Click **Deploy new agent**
5. Select **Windows** (should auto-detect)
6. In **Agent name**, enter: `windows-victim`
7. In **Server address**, enter: `192.168.100.10` (the Wazuh Server IP)
8. Click **Generate installer**

The dashboard will generate an installer command. Copy this entire command.

### 5.2 Run Agent Installer on Windows VM

1. In the **Windows VM**, open **PowerShell as Admin**
2. Paste the command generated by Wazuh dashboard (will look like):
```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.14/windows/wazuh-agent-4.14.0-1.msi -OutFile wazuh-agent.msi; msiexec.exe /i wazuh-agent.msi /q WAZUH_MANAGER='192.168.100.10' WAZUH_REGISTRATION_SERVER='192.168.100.10' WAZUH_AGENT_NAME='windows-victim'
```

3. Press **Enter**
4. The installer will run silently. You may see a brief UAC prompt — allow it.
5. When complete, you'll see no output (normal for silent install).

### 5.3 Start Wazuh Agent Service

In **PowerShell as Admin**:

```powershell
Start-Service WazuhSvc
```

Verify the agent is running:

```powershell
Get-Service WazuhSvc | Select-Object Status
```

Should show: `Running`

### 5.4 Verify Agent is Connected in Dashboard

1. Go back to Wazuh Dashboard: `https://192.168.100.10`
2. Click **Agents**
3. You should see **windows-victim** listed with status **Active** in green

The dashboard shows:
```
Agent ID: 002 (or higher)
Agent Name: windows-victim
IP: 192.168.100.20
Status: Active
```

---

## Step 6: Verify Log Collection

### 6.1 Generate Test Event

Cause Windows to generate a security event:

1. In **Windows VM**, open **PowerShell as Admin**
2. Try to run an admin command a few times (to trigger failed UAC):
```powershell
Get-EventLog -LogName Security -Newest 1
```

3. Or try to log in with a wrong password a few times (generates "Failed Logon" events)

### 6.2 Check Logs in Wazuh Dashboard

1. Go to **Wazuh Dashboard** → **Security Events**
2. Look for events from the **windows-victim** agent
3. You should see Windows event logs, Sysmon events, or process execution alerts

If you see events, congratulations! The agent is successfully collecting and sending logs.

### 6.3 Check Agent Logs (if no events appear)

If the dashboard shows the agent as Active but no events appear, check the agent logs:

In **Windows VM Command Prompt (Admin)**:

```cmd
cd "C:\Program Files (x86)\ossec-agent"
type ossec.log
```

Look for entries like:
- `wazuh: Connected` — agent connected to manager
- `wazuh: Received response` — agent communicating

If you see errors, the agent may not be configured correctly. Re-run the installer command from Step 5.2.

---

## Troubleshooting

### Issue: Agent shows "Disconnected" in dashboard
**Solution**:
1. Verify network connectivity:
```cmd
ping 192.168.100.10
```

2. Restart agent service:
```powershell
Restart-Service WazuhSvc
```

3. Check Windows Firewall is off (Step 4.4)
4. Check agent logs for errors (Step 6.3)

### Issue: No logs appearing in dashboard despite agent being Active
**Solution**:
1. Verify agent is collecting events:
```cmd
wevtutil qe Security /c:10
```

2. Check if audit policies are enabled:
```powershell
auditpol /get /category:*
```

3. Restart the agent:
```powershell
Restart-Service WazuhSvc
```

4. Wait 2-3 minutes for logs to index, then refresh dashboard

### Issue: Agent installation fails or uninstalls immediately
**Solution**:
1. Ensure you're running PowerShell as Admin
2. Check that the server address (192.168.100.10) is correct
3. Verify the Wazuh Server is running (check Phase 2)
4. Try manual installation (see Alternative Installation below)

### Alternative Installation (Manual)

If the automatic method fails:

1. Download MSI file manually:
```powershell
cd C:\Temp
Invoke-WebRequest -Uri https://packages.wazuh.com/4.14/windows/wazuh-agent-4.14.0-1.msi -OutFile wazuh-agent.msi
```

2. Edit `C:\Program Files (x86)\ossec-agent\ossec.conf` and set:
```xml
<client>
  <server ip="192.168.100.10">
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

3. Restart the service:
```powershell
Restart-Service WazuhSvc
```

---

## What Happens Next

After completing this phase:
1. ✅ Windows 10 VM is running with static IP 192.168.100.20
2. ✅ Wazuh Agent installed and connected (shows Active in dashboard)
3. ✅ Windows logs being collected in real time
4. ✅ You can see security events in the Wazuh Dashboard

Next: **Phase 4: Kali Attacker VM Setup** — where you'll create the red team machine that attacks this victim.

---

## Verification Checklist

- [ ] Windows 10 VM created with correct specs (2 vCPU, 4GB RAM, 60GB disk, VMnet2)
- [ ] Windows 10 installed and booted successfully
- [ ] Static IP set to 192.168.100.20 (verified with `ipconfig /all`)
- [ ] Can ping Wazuh Server at 192.168.100.10
- [ ] Windows Event Logging enabled (audit policies configured)
- [ ] Windows Defender enabled (optional but recommended)
- [ ] Sysmon installed with SwiftOnSecurity config (optional but recommended)
- [ ] Windows Firewall disabled (for lab simplicity)
- [ ] Wazuh Agent downloaded and installed
- [ ] Agent service WazuhSvc is running
- [ ] Agent appears in Wazuh Dashboard as "Active"
- [ ] Agent name shows as "windows-victim"
- [ ] Dashboard shows agent IP as 192.168.100.20
- [ ] Security events appearing in Wazuh Dashboard (from agent)
- [ ] Understand this VM is the "blue team's endpoint" to protect
- [ ] Ready to create attacker VM for red team simulations

---

## Reference Commands Quick List

```cmd
# Check IP configuration
ipconfig /all

# Test connectivity to Wazuh Server
ping 192.168.100.10

# Check Wazuh Agent service status
Get-Service WazuhSvc

# Restart Wazuh Agent
Restart-Service WazuhSvc

# View Wazuh Agent logs
cd "C:\Program Files (x86)\ossec-agent"
type ossec.log

# Enable audit policies (PowerShell Admin)
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
auditpol /set /subcategory:"Account Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Logon/Logoff" /success:enable /failure:enable

# View recent Windows security events
wevtutil qe Security /c:20
```

---

## Next Phase
Proceed to **Phase 4: Kali Attacker VM Setup** when all checklist items are complete.
