# Phase 6: Analyze and Build Custom Rules

## Overview
This is the **advanced blue team phase** where you move from simply *detecting* attacks to *understanding* them and *improving* your detection rules. You'll:
- Analyze alerts in detail
- Understand how detection rules work
- Create custom rules for attacks Wazuh missed
- Build custom dashboards
- Set up automated responses
- Learn MITRE ATT&CK framework integration

This is what real SOC analysts do — constantly tuning their detection systems.

---

## Part 1: Mastering the Wazuh Dashboard

### 1.1 Understanding Alert Details

When you see an alert in the dashboard, clicking it shows rich information:

1. Go to **Security Events**
2. Click any alert to expand it
3. You'll see multiple sections:

**Top Section (Alert Summary):**
- **Rule ID**: Unique identifier (e.g., 40101)
- **Rule Name**: Human-readable description
- **Level**: Severity from 1-15
- **Timestamp**: When it occurred

**Middle Section (Event Details):**
- **Source IP**: Where attack came from
- **Destination IP**: Target system
- **User**: Windows user involved
- **Process Name**: What program triggered alert
- **Command Line**: What command was run

**Bottom Section (Raw Log Data):**
- Original Windows event log entry
- Full event data in XML format

### 1.2 Filtering and Searching Alerts

In **Security Events**, use filters to find specific attacks:

**By Agent:**
1. Click **Agents** dropdown
2. Select your agent (e.g., "windows-victim")
3. View only events from that agent

**By Rule ID:**
1. Click **Filters** (top of table)
2. Add filter: `rule.id` equals `18151`
3. Now see only failed login attempts

**By Severity:**
1. Add filter: `rule.level` greater than `10`
2. Now see only high/critical events

**By IP Address:**
1. Add filter: `source_ip` equals `192.168.100.30`
2. Now see only events from Kali attacker

**By Time Range:**
1. Click **Last 24 hours** (top right)
2. Select **Last 7 days** or custom range
3. View alerts over different time periods

### 1.3 Creating Dashboard Panels

Build a custom view of important data:

1. Go to **Dashboards** → **Create Dashboard**
2. Click **Add widget**
3. Choose visualization type:
   - **Bar chart**: Attacks by type
   - **Pie chart**: Events by severity
   - **Table**: Detailed event list
   - **Timeline**: Events over time
   - **Geo map**: Attack origins by location

**Example: Create "Attack Timeline" Dashboard:**
1. Add Time-series chart widget
2. Field: `rule.name`
3. Aggregation: Count
4. Time interval: 1 hour
5. Title: "Security Events Per Hour"

This shows when attacks occurred and helps identify attack windows.

**Example: Create "Top Rules Fired" Dashboard:**
1. Add Bar chart widget
2. Metric: Count of events
3. X-axis: `rule.name`
4. This shows which detections are most triggered

### 1.4 Viewing the MITRE ATT&CK Framework

Wazuh automatically maps alerts to MITRE ATT&CK tactics:

1. Go to **Threat hunting** → **MITRE ATT&CK**
2. Select a **tactic** (e.g., "Initial Access", "Privilege Escalation")
3. See all techniques under that tactic
4. See which rules detect each technique

This helps you understand:
- What attack phase (reconnaissance → exploitation → persistence)
- What detection gaps exist
- Which techniques are well-covered vs. uncovered

---

## Part 2: Understanding Wazuh Detection Rules

### 2.1 Rule File Location

Rules are stored on the **Wazuh Server** at:

```
/var/ossec/etc/rules/
```

Default rules:
```
/var/ossec/etc/rules/windows_events.xml
/var/ossec/etc/rules/sysmon_rules.xml
/var/ossec/etc/rules/auth_rules.xml
```

Custom rules go in:
```
/var/ossec/etc/rules/local_rules.xml
```

### 2.2 Rule Structure (XML Format)

Here's a simple rule example:

```xml
<group name="brute_force">
  <rule id="18151" level="7">
    <if_sid>18100</if_sid>
    <match>Failure Audit</match>
    <description>Failed login attempt</description>
  </rule>
</group>
```

Breaking it down:

```xml
<rule id="18151" level="7">
```
- `id`: Unique identifier (must not conflict with existing)
- `level`: Severity 1-15 (7 = high)

```xml
<if_sid>18100</if_sid>
```
- Parent rule to inherit from
- This rule fires when parent (18100) matches

```xml
<match>Failure Audit</match>
```
- Pattern to match in the log
- Case-sensitive
- Matches "Failure Audit" in Windows events

```xml
<description>Failed login attempt</description>
```
- Human-readable description (appears in dashboard)

### 2.3 Rule Logic — Decoders vs. Rules

**Decoders** (decode logs into structured data):
- Input: Raw Windows event log
- Output: Structured fields (source, destination, user, etc.)
- Example: decoder extracts "Administrator" from "User: Administrator"

**Rules** (detect based on decoded fields):
- Input: Decoded fields from decoder
- Condition: Match pattern, threshold, time-based logic
- Output: Alert with rule ID and severity

### 2.4 Common Rule Types

**Pattern Matching:**
```xml
<match>ATTACK PATTERN HERE</match>
```
Alerts when text contains this pattern.

**Threshold-based:**
```xml
<frequency>10</frequency>
<timeframe>600</timeframe>
```
Alerts when pattern occurs 10+ times in 600 seconds (10 minutes).

**Composite Rules:**
```xml
<if_sid>18150, 18151, 18152</if_sid>
```
Alerts when any of these rule IDs fire (correlate multiple events).

**Field-based:**
```xml
<field name="win.eventid">^(4688|4689)$</field>
```
Alerts when Windows event ID is 4688 or 4689 (process creation).

---

## Part 3: Creating Custom Detection Rules

### 3.1 Identify a Detection Gap

During Phase 5, did Wazuh miss any attacks you performed? Maybe:
- A suspicious file extension (.exe in C:\Temp)
- A specific hacking tool (you ran `netstat` with unusual flags)
- A custom script pattern
- A persistence mechanism Wazuh didn't detect

This is your gap — create a rule to fill it.

### 3.2 Create a Custom Rule File

**On the Wazuh Server**, create the custom rules file:

```bash
sudo nano /var/ossec/etc/rules/local_rules.xml
```

If the file doesn't exist, start with this template:

```xml
<!-- Custom detection rules for SOC Lab -->
<!-- Filename: /var/ossec/etc/rules/local_rules.xml -->
<group name="soc_lab_custom">

  <!-- Rule 100001: Detect suspicious .exe in temp directories -->
  <rule id="100001" level="9">
    <if_sid>3401,3402,3403</if_sid>
    <field name="win.eventdata.targetFilename">\.(exe|dll|bat|cmd|ps1)$</field>
    <regex>\\(Temp|AppData|Download)</regex>
    <description>Suspicious executable in temp directory</description>
  </rule>

  <!-- Rule 100002: Detect multiple failed admin commands -->
  <rule id="100002" level="8">
    <match>Access is denied</match>
    <frequency>5</frequency>
    <timeframe>300</timeframe>
    <description>Multiple failed admin command attempts (possible escalation)</description>
  </rule>

  <!-- Rule 100003: Detect suspicious PowerShell usage -->
  <rule id="100003" level="10">
    <field name="win.eventdata.commandLine">powershell.*(-Enc|-EncodedCommand|-ExecutionPolicy.*Bypass|-NoP)</field>
    <description>Suspicious PowerShell command with encoding or bypass</description>
  </rule>

  <!-- Rule 100004: Detect usage of hacking tools -->
  <rule id="100004" level="8">
    <field name="win.eventdata.image">(nmap|hashcat|hydra|mimikatz|psexec)</field>
    <description>Known hacking tool detected on Windows system</description>
  </rule>

  <!-- Rule 100005: Detect attempts to disable Windows Defender -->
  <rule id="100005" level="11">
    <match>Defender Malware Protection</match>
    <match>was disabled</match>
    <description>Windows Defender disabled - possible evasion</description>
  </rule>

</group>
```

### 3.3 Explanation of Custom Rules

**Rule 100001 — Suspicious Executables in Temp:**
```xml
<regex>\\(Temp|AppData|Download)</regex>
```
Detects .exe/.dll files in Temp directories (common malware staging area).

**Rule 100002 — Brute Force Escalation:**
```xml
<frequency>5</frequency>
<timeframe>300</timeframe>
```
Fires when "Access is denied" appears 5+ times in 300 seconds (failed admin attempts).

**Rule 100003 — PowerShell Obfuscation:**
```xml
<field name="win.eventdata.commandLine">powershell.*(-Enc|-EncodedCommand|...)</field>
```
Detects PowerShell with encoding flags (common malware delivery).

**Rule 100004 — Hacking Tools:**
```xml
<field name="win.eventdata.image">(nmap|hashcat|hydra|...)</field>
```
Simple but effective — alerts if these tool names appear in process names.

**Rule 100005 — Defender Bypass:**
```xml
<match>was disabled</match>
```
Detects when Windows Defender is disabled (evasion tactic).

### 3.4 Save and Validate Rules

Save the file:
```bash
# Ctrl+X, Y, Enter to save in nano
```

Validate syntax:
```bash
sudo /var/ossec/bin/wazuh-control info
```

### 3.5 Reload Wazuh to Apply Rules

```bash
sudo systemctl restart wazuh-manager
```

Or use:
```bash
sudo /var/ossec/bin/wazuh-control restart
```

Wait 10-20 seconds for restart.

### 3.6 Test Your Custom Rules

Generate events that should match your rules:

**Test Rule 100001 (temp .exe):**
```powershell
# On Windows Victim
Copy-Item C:\Windows\notepad.exe C:\Temp\suspicious.exe
```

**Test Rule 100003 (encoded PowerShell):**
```powershell
# Encoded command (IEX Invoke-Expression)
powershell -Enc "V3JpdGUtSG9zdCAiSGVsbG8gV29ybGQi"
```

**Test Rule 100004 (hacking tools):**
```powershell
# Create dummy nmap process entry (or actually run nmap against Windows)
# From Kali: nmap 192.168.100.20 (Windows logs this as network connection)
```

Check dashboard — your new custom rules should fire!

---

## Part 4: Advanced Rule Techniques

### 4.1 Correlation Rules (Multi-Event Detection)

Detect attack campaigns by correlating multiple events:

```xml
<!-- Rule 100010: Brute force followed by successful logon -->
<rule id="100010" level="12">
  <if_sid>18151,4624</if_sid>
  <frequency>10</frequency>
  <timeframe>1800</timeframe>
  <same_field>source_ip</same_field>
  <description>Multiple failed logins followed by success - possible compromise</description>
</rule>
```

This rule:
1. Watches for failed logins (18151) AND successful logins (4624)
2. From the SAME IP address
3. Within 30 minutes (1800 seconds)
4. If pattern occurs 10+ times = alert

### 4.2 Regular Expression Patterns

Match complex patterns:

```xml
<!-- Detect PowerShell reverse shell patterns -->
<rule id="100011" level="11">
  <field name="win.eventdata.commandLine">powershell.*(New-Object.*socket|System.Net.Sockets.*TcpClient)</field>
  <description>PowerShell reverse shell detected</description>
</rule>
```

```xml
<!-- Detect base64 encoded downloads -->
<rule id="100012" level="10">
  <field name="win.eventdata.commandLine">(\-[eE]ncodedCommand\s+[A-Za-z0-9+/]{20,}|IEX.*DownloadString)</field>
  <description>Suspicious base64 encoded command execution</description>
</rule>
```

### 4.3 Rules Using Mitre ATT&CK Mapping

Add mitre mapping to your rules:

```xml
<rule id="100013" level="11">
  <field name="win.eventdata.commandLine">reg.*add.*run</field>
  <mitre>
    <technique id="T1547.001">Boot or Logon Autostart Execution - Registry Run Keys</technique>
    <tactic>Persistence</tactic>
  </mitre>
  <description>Registry run key modified for persistence</description>
</rule>
```

Now your custom rules appear in the MITRE ATT&CK dashboard view!

---

## Part 5: Setting Up Automated Responses

### 5.1 Email Alerts for Critical Events

Configure Wazuh to email you when critical alerts fire:

**On Wazuh Server**, edit the config:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add this section (find the `<alerting>` section):

```xml
<email_notification>yes</email_notification>
<email_to>your-email@gmail.com</email_to>
<smtp_server>smtp.gmail.com</smtp_server>
<smtp_port>587</smtp_port>
<email_from>wazuh-server@lab.local</email_from>
<email_maxpersec>10</email_maxpersec>
```

For **rules to trigger emails**, they need `level >= 9`:

```xml
<!-- Email will be sent for this rule (level 11) -->
<rule id="100013" level="11">
  <email_alert>yes</email_alert>
  ...
</rule>
```

Restart Wazuh:
```bash
sudo systemctl restart wazuh-manager
```

### 5.2 Webhook Alerts (Slack, Teams, etc.)

Send alerts to Slack when attacks occur:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add webhook configuration:

```xml
<integration>
  <name>slack</name>
  <hook_url>https://hooks.slack.com/services/YOUR/WEBHOOK/URL</hook_url>
  <level>7</level>
  <group>^(access_denied|brute_force)$</group>
  <alert_format>json</alert_format>
</integration>
```

Now critical alerts post directly to your Slack channel!

### 5.3 Active Response (Automatic Blocking)

Have Wazuh automatically block attacking IPs:

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Find `<active-response>` section and add:

```xml
<active-response>
  <command>firewall-drop</command>
  <location>agent</location>
  <level>7</level>
  <timeout>600</timeout>
  <rules>18151,18152,40101</rules>
</active-response>
```

This tells Wazuh:
- When rules 18151/18152/40101 fire (brute force, recon)
- Execute the `firewall-drop` command
- On the affected agent (Windows victim)
- For 600 seconds (10 minutes)
- Block the attacker's IP address

Now when Kali attacks Windows, it gets blocked after 3-5 failed attempts!

---

## Part 6: Incident Response Workflow

### 6.1 Investigating an Alert

When a critical alert appears, follow this workflow:

**Step 1: Identify the Attack**
```
Alert: Rule 3105 - PowerShell suspicious activity
Level: 11 (Critical)
Time: 2024-01-15 14:32:10
Agent: windows-victim
Source IP: 192.168.100.30
```

**Step 2: Understand What Happened**
Click alert → view details:
- What command was run?
- By which user?
- On what process?
- What was the result?

Example:
```
Command: powershell -Enc "V3JpdGUtSG9zdCAi..."
User: Administrator
Parent Process: explorer.exe
Result: Success
```

**Step 3: Analyze the Indicator of Compromise (IOC)**
- Source IP: 192.168.100.30 (attacker's Kali machine)
- Process: powershell.exe
- Command pattern: `-Enc` flag (encoding — red flag)
- User: Administrator (running with high privileges)

**Step 4: Check for Related Alerts**
Filter: `source_ip = 192.168.100.30` and `level > 8`

This shows the full attack timeline:
```
14:30:00 - Brute force attempt (Rule 18151) ← authentication attack
14:31:00 - Failed logons (Rule 18152)       ← failed multiple times
14:32:00 - Process execution (Rule 3105)    ← got in, running code
14:33:00 - File modification (Rule 3401)    ← moving laterally
14:35:00 - Scheduled task created (Rule 3602) ← persistence
```

**Step 5: Take Action**
- Block the IP (manual or via active response)
- Isolate the infected machine
- Notify the system owner
- Start detailed forensics (on an offline copy)
- Review firewall/access logs
- Check for data exfiltration

### 6.2 False Positive Management

Not all alerts are real attacks. Learn to identify false positives:

**False Positive Example:**
- Admin legitimately copies system files to C:\Temp for backup
- Rule 100001 fires (exe in temp)
- But it's authorized activity

**Solution:** Tune your rule:
```xml
<!-- Original (generates false positives) -->
<rule id="100001" level="9">
  <field name="win.eventdata.targetFilename">\.(exe|dll)$</field>
  <regex>\\Temp</regex>
</rule>

<!-- Tuned (excludes backup processes) -->
<rule id="100001" level="9">
  <field name="win.eventdata.targetFilename">\.(exe|dll)$</field>
  <regex>\\Temp</regex>
  <not_match>backup</not_match> <!-- Exclude backups -->
</rule>
```

Or create an exception rule:
```xml
<rule id="100100" level="0">
  <if_sid>100001</if_sid>
  <user>administrator</user>
  <field name="win.eventdata.image">backup.exe</field>
  <description>Legitimate backup activity - ignore</description>
</rule>
```

---

## Part 7: Building a Compliance Dashboard

Create a dashboard for compliance reporting (HIPAA, PCI-DSS, etc.):

### 7.1 Create Compliance Dashboard

1. Go to **Dashboards** → **Create Dashboard**
2. Name it: "Compliance Report - January 2024"
3. Add these widgets:

**Widget 1: Failed Login Attempts**
```
Chart Type: Time series
Metric: Count of Event ID 4625
Filter: Event ID = 4625
Purpose: Monitor failed authentication (HIPAA requirement)
```

**Widget 2: File Access to Sensitive Data**
```
Chart Type: Table
Fields: timestamp, user, file_path, action
Filter: file_path contains "confidential|secret|PII"
Purpose: Track access to sensitive data (PCI-DSS requirement)
```

**Widget 3: Privilege Elevation Events**
```
Chart Type: Bar chart
Metric: Count by user
Filter: Rule ID = 18103, 18104
Purpose: Monitor privilege escalation (SOX compliance)
```

**Widget 4: System Configuration Changes**
```
Chart Type: Timeline
Events: Registry modifications, Service changes, Firewall changes
Purpose: Track system integrity (HIPAA technical safeguards)
```

### 7.2 Export Compliance Reports

1. Go to **Dashboards** → Your compliance dashboard
2. Click **Export** (top right)
3. Select **PDF** format
4. Email to compliance team

This shows you've been monitoring for threats!

---

## Part 8: Threat Hunting with Wazuh

Go beyond reactive detection — actively hunt for threats:

### 8.1 Hunting for Lateral Movement

Query: "Who accessed network shares on other systems?"

In **Queries** section:
```
agent.name: windows-victim
event.code: (5140 OR 5145)  # Network share accessed
action.type: "access"
severity: 5
```

Results show all unauthorized share access attempts.

### 8.2 Hunting for Credential Theft

Query: "Has anyone dumped credentials or accessed LSASS?"

```
process.name: lsass.exe
event.action: "accessed"
severity: 10
```

LSASS (Local Security Authority Subsystem Service) stores credentials. If accessed = credential theft attempt.

### 8.3 Hunting for Persistence Mechanisms

Query: "Find all persistence mechanisms (scheduled tasks, run keys, etc.)"

```
(rule.name: "Scheduled task" OR "Registry run key" OR "Service installed")
agent.name: windows-victim
```

Shows all persistence attempts, even if obfuscated.

---

## Quick Reference: Rule Writing Syntax

### Common Operators

```xml
<!-- Match text (case-sensitive) -->
<match>ADMIN_AUDIT</match>

<!-- Regular expression -->
<field name="field_name">regex_pattern</field>

<!-- Threshold detection -->
<frequency>10</frequency>          <!-- Trigger when occurs 10+ times -->
<timeframe>600</timeframe>          <!-- Within 600 seconds -->

<!-- Correlation (multiple events) -->
<if_sid>rule1,rule2,rule3</if_sid>

<!-- Parent/child rules -->
<if_sid>4001</if_sid>

<!-- Field matching -->
<field name="win.eventid">^(4688|4689)$</field>

<!-- Negation -->
<not_match>legitimate_process</not_match>
```

### Common Fields

```xml
<!-- Windows Event Log Fields -->
<field name="win.eventid">4625</field>                    <!-- Failed logon -->
<field name="win.eventdata.user">administrator</field>    <!-- Username -->
<field name="win.eventdata.image">C:\Windows\cmd.exe</field>  <!-- Process -->
<field name="win.eventdata.commandLine">-exec bypass</field>  <!-- Command -->
<field name="win.eventdata.targetFilename">C:\temp\</field>   <!-- File path -->
<field name="win.eventdata.sourceIp">192.168.1.100</field>    <!-- Network source -->

<!-- Standard Fields -->
<field name="source_ip">192.168</field>
<field name="agent.name">windows-victim</field>
<field name="rule.level">7</field>
```

### Severity Levels

```
Level 1-2:   Debug / Informational
Level 3-4:   Minimal / Low
Level 5-6:   Medium / Warning
Level 7-8:   High
Level 9-11:  Very High / Critical
Level 12-15: Extremely Critical (investigation required)
```

---

## What You've Accomplished

After Phase 6, you've:
1. ✅ Analyzed security alerts in detail
2. ✅ Understood Wazuh rule structure and logic
3. ✅ Created custom detection rules
4. ✅ Set up automated responses and email alerts
5. ✅ Built compliance dashboards
6. ✅ Performed incident response investigations
7. ✅ Hunted for threats proactively
8. ✅ Identified and fixed false positives

**You've built a fully-functional SOC lab.**

---

## Verification Checklist

- [ ] Can navigate Wazuh Dashboard security events
- [ ] Can filter and search alerts by various criteria
- [ ] Understand rule ID, level, and description meanings
- [ ] Created at least one custom detection rule
- [ ] Custom rule file created at /var/ossec/etc/rules/local_rules.xml
- [ ] Custom rules validated and Wazuh restarted
- [ ] Custom rule triggered and appeared in dashboard
- [ ] Understand MITRE ATT&CK mapping
- [ ] Set up email alerts for critical events
- [ ] Understand correlation rules (multi-event detection)
- [ ] Practiced incident response workflow
- [ ] Created at least one compliance dashboard
- [ ] Tested threat hunting queries
- [ ] Understand false positive identification and tuning
- [ ] Know how to use regex in rules
- [ ] Understand severity levels and thresholds
- [ ] Can explain attack-to-detection pipeline to others
- [ ] Ready for real-world SOC work or certifications

---

## Next Steps (Beyond the Lab)

### Continue Learning:
1. **Study MITRE ATT&CK framework** - Deep dive into attack techniques
2. **Learn threat intelligence** - Research real CVEs and exploits
3. **Practice on CTF platforms** - HackTheBox, TryHackMe
4. **Setup SIEM in enterprise** - Deploy Wazuh in production environment
5. **Pursue certifications** - CEH, OSCP, GCIA, GCIH

### Enhance Your Lab:
1. Add more VMs (web server, domain controller, file server)
2. Simulate multi-stage attacks (APT simulations)
3. Integrate threat intelligence feeds (AlienVault OTX, Shodan)
4. Set up file integrity monitoring on critical files
5. Implement deception (honeypots, honeypots files)
6. Create ransomware simulation scenarios

### Tools to Add:
1. **TheHive** - Incident response platform
2. **Splunk** - Alternative SIEM (free tier available)
3. **osquery** - System monitoring
4. **Suricata** - Network IDS/IPS
5. **Zeek** - Network traffic analysis

---

## Final Summary: Your SOC Lab Architecture

```
┌─────────────────────── VMware Workstation ───────────────────────┐
│                                                                    │
│  Host-Only Network: 192.168.100.0/24                             │
│                                                                    │
│  ┌──────────────────────┐  ┌──────────────────┐  ┌────────────┐ │
│  │  Wazuh Server        │  │ Windows Victim   │  │ Kali       │ │
│  │  192.168.100.10      │  │ 192.168.100.20   │  │ Attacker   │ │
│  │                      │  │                  │  │ .30        │ │
│  │ - Manager            │  │ - Wazuh Agent    │  │            │ │
│  │ - Indexer (OpenSearch)  │ - Windows Logs   │  │ - nmap     │ │
│  │ - Dashboard (Web UI) │  │ - Sysmon         │  │ - hydra    │ │
│  │                      │  │ - FIM enabled    │  │ - metasploit│ │
│  │ Receives all logs ←─────┤ Sends all data   │  │ - hashcat  │ │
│  │ Detects attacks      │  │ to manager       │  │            │ │
│  │ Alerts and responds  │  │                  │  │ Performs   │ │
│  │                      │  │ Attack target ←──────┤ attacks    │ │
│  └──────────────────────┘  └──────────────────┘  └────────────┘ │
│                                                                    │
│  WORKFLOW:                                                         │
│  1. Kali attacks Windows                                          │
│  2. Windows logs attack → Wazuh Agent → Manager                  │
│  3. Manager analyzes log against detection rules                  │
│  4. Matching rule fires → Alert in Dashboard                     │
│  5. Blue team (SOC analyst) investigates alert                   │
│  6. Response (block IP, isolate machine, etc.)                   │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

You're now equipped to:
- Detect sophisticated attacks in real-time
- Build custom detection rules
- Investigate security incidents
- Set up automated responses
- Work in a real SOC environment

---

## References & Further Reading

### Official Documentation:
- Wazuh Docs: https://documentation.wazuh.com/
- MITRE ATT&CK Framework: https://attack.mitre.org/

### Tools Used:
- Wazuh: https://www.wazuh.com/
- Kali Linux: https://www.kali.org/
- Sysmon: https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon
- Metasploit: https://www.metasploit.com/

### Certifications to Pursue:
- CompTIA Security+
- CEH (Certified Ethical Hacker)
- GIAC GCIA (Certified Intrusion Analyst)
- eLearnSecurity eCDFP (Certified Defensive Security Professional)

---

## Congratulations! 🎉

You've completed a comprehensive SOC lab demonstrating Blue Team and Red Team skills. You understand:
- ✅ Offensive security (how attackers operate)
- ✅ Defensive security (how to detect attacks)
- ✅ SIEM platforms (Wazuh)
- ✅ Detection engineering (creating rules)
- ✅ Incident response (investigating alerts)

**You're ready for your first SOC analyst role!**

---

## Verification Checklist (Final)

- [ ] Completed Phase 1-5 successfully
- [ ] Created custom detection rules in local_rules.xml
- [ ] Dashboard shows custom rule alerts
- [ ] Understand full attack detection pipeline
- [ ] Can investigate security incidents
- [ ] Can create and tune detection rules
- [ ] Understand MITRE ATT&CK framework
- [ ] Set up automated responses/alerts
- [ ] Created compliance dashboard
- [ ] Know next steps for continued learning
- [ ] Ready to apply for SOC analyst positions
- [ ] Can explain this lab in a job interview

---

## Contact & Support

**Issues with Wazuh?**
- Official Slack: https://wazuh.com/community/slack/
- GitHub Issues: https://github.com/wazuh/wazuh/issues
- Documentation: https://documentation.wazuh.com/

**Expand Your Skills:**
- Practice on HackTheBox: https://www.hackthebox.com/
- Practice on TryHackMe: https://www.tryhackme.com/
- Join CTF competitions: https://ctftime.org/

---

**End of Phase 6 - Congratulations on Completing Your SOC Lab!**
