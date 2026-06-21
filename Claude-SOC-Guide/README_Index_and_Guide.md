# Wazuh SOC Lab - Complete Project Guide
## Index & Quick Reference

---

## 📋 Project Overview

**Goal**: Build a complete Security Operations Center (SOC) lab with Wazuh to simulate real-world threat detection and response.

**Architecture**: 
- **Blue Team HQ**: Wazuh Server (Manager, Indexer, Dashboard)
- **Victim Endpoint**: Windows 10 with Wazuh Agent
- **Red Team**: Kali Linux with offensive tools
- **Network**: Isolated VMware Host-Only Network (192.168.100.0/24)

**Your Role**: Both red team (attacker) and blue team (defender) to understand the full attack-to-detection lifecycle.

**Duration**: 8-14 days depending on your pace

**Difficulty**: Intermediate (assumes basic networking and Linux/Windows knowledge)

---

## 📁 File Structure

This project consists of **6 comprehensive markdown files**:

```
├── Phase_1_VMware_Network_Setup.md           (~5-6 hours)
├── Phase_2_Wazuh_Server_VM.md                (~8-12 hours)
├── Phase_3_Windows_Victim_VM.md              (~4-6 hours)
├── Phase_4_Kali_Attacker_VM.md               (~3-5 hours)
├── Phase_5_Simulate_and_Detect_Threats.md    (~6-10 hours)
└── Phase_6_Analyze_and_Build_Custom_Rules.md (~4-8 hours)
```

---

## 🚀 Quick Start Checklist

### Day 1-2: Infrastructure Setup
- [ ] Read Phase 1: VMware Network Setup
- [ ] Create Host-Only network (VMnet2) with subnet 192.168.100.0/24
- [ ] Verify network configuration

### Day 3-4: Deploy Wazuh Server
- [ ] Read Phase 2: Wazuh Server VM Setup
- [ ] Create Ubuntu 22.04 VM (4 vCPU, 8GB RAM, 100GB disk)
- [ ] Install Wazuh all-in-one (Manager, Indexer, Dashboard)
- [ ] Access dashboard at https://192.168.100.10
- [ ] Verify all services running

### Day 5-6: Deploy Victim Machine
- [ ] Read Phase 3: Windows 10 Victim VM Setup
- [ ] Create Windows 10 VM (2 vCPU, 4GB RAM, 60GB disk)
- [ ] Install Wazuh Agent
- [ ] Confirm agent shows "Active" in dashboard
- [ ] Verify Windows event logging enabled

### Day 7: Deploy Attacker Machine
- [ ] Read Phase 4: Kali Attacker VM Setup
- [ ] Create Kali Linux VM (2 vCPU, 4GB RAM, 40GB disk)
- [ ] Verify all offensive tools installed (nmap, hydra, metasploit)
- [ ] Verify network connectivity to victim and server

### Day 8-10: Conduct Attacks & Detect
- [ ] Read Phase 5: Simulate and Detect Threats
- [ ] Launch reconnaissance scan (nmap)
- [ ] Watch alerts appear in Wazuh Dashboard
- [ ] Conduct brute force attack (hydra)
- [ ] Simulate malware execution (PowerShell)
- [ ] Observe FIM alerts on file modifications
- [ ] Try metasploit exploitation

### Day 11-14: Tune Detection & Response
- [ ] Read Phase 6: Analyze and Build Custom Rules
- [ ] Create custom detection rules
- [ ] Set up email/webhook alerts
- [ ] Build compliance dashboards
- [ ] Practice incident response workflow
- [ ] Identify and tune false positives

---

## 💾 Hardware Requirements

### System Requirements
- **Host PC CPU**: Ryzen 5 Hexacore or better
- **Host PC RAM**: 24 GB (minimum 16 GB)
- **Host PC Storage**: 500 GB SSD (minimum 300 GB)
- **Virtualization**: VMware Workstation Pro/Player

### VM Allocation
```
Wazuh Server:     4 vCPU | 8 GB RAM  | 100 GB disk
Windows Victim:   2 vCPU | 4 GB RAM  | 60 GB disk
Kali Attacker:    2 vCPU | 4 GB RAM  | 40 GB disk
─────────────────────────────────────────────────
Total:           8 vCPU | 16 GB RAM | 200 GB disk
Host OS Reserve:        | 8 GB RAM  | 300 GB disk
```

---

## 🔧 Troubleshooting Index

### Common Issues & Solutions

| Issue | Phase | Solution |
|-------|-------|----------|
| Network not isolated | 1 | Recreate Host-Only network, verify DHCP disabled |
| Wazuh fails to install | 2 | Check disk space, verify Ubuntu 22.04 LTS, check internet |
| Agent won't connect | 3 | Verify firewall off, check IP config, restart agent service |
| Can't ping between VMs | 1,3,4 | Verify all VMs on VMnet2, check firewall settings |
| No alerts appearing | 3,5 | Verify agent active, enable audit policies, generate test events |
| Custom rules not working | 6 | Check XML syntax, verify rule IDs unique, restart wazuh-manager |

---

## 📚 Learning Objectives

### By Phase 1: Network Fundamentals
- [ ] Understand VMware virtual networking
- [ ] Know difference between NAT, Bridged, and Host-Only
- [ ] Can configure static IP addresses
- [ ] Understand IP subnetting (CIDR notation)

### By Phase 2: SIEM Architecture
- [ ] Understand what a SIEM is and why it matters
- [ ] Know the role of Manager, Indexer, and Dashboard
- [ ] Understand logs vs. events vs. alerts
- [ ] Can navigate Wazuh Dashboard

### By Phase 3: Endpoint Security
- [ ] Understand agent-based monitoring
- [ ] Know Windows event logs
- [ ] Understand FIM (File Integrity Monitoring)
- [ ] Can troubleshoot agent connectivity

### By Phase 4: Offensive Security
- [ ] Know reconnaissance techniques (nmap)
- [ ] Understand authentication attacks (brute force)
- [ ] Know exploitation frameworks (metasploit)
- [ ] Understand attacker methodology

### By Phase 5: Detection Engineering
- [ ] Understand attack-to-detection pipeline
- [ ] Know MITRE ATT&CK framework
- [ ] Can correlate multiple events into campaigns
- [ ] Can map attacks to detections

### By Phase 6: Incident Response
- [ ] Can investigate security incidents
- [ ] Create custom detection rules
- [ ] Set up automated responses
- [ ] Build compliance dashboards
- [ ] Understand false positive tuning

---

## 🎓 What You'll Be Able to Do

### Red Team Skills
- ✅ Perform network reconnaissance (port scanning, service enumeration)
- ✅ Attempt unauthorized access (brute force, exploitation)
- ✅ Execute malicious code (PowerShell attacks, reverse shells)
- ✅ Establish persistence (scheduled tasks, registry modifications)
- ✅ Simulate lateral movement and data exfiltration

### Blue Team Skills
- ✅ Detect reconnaissance activities in real-time
- ✅ Alert on authentication attacks
- ✅ Detect malware execution and suspicious behavior
- ✅ Monitor file system integrity
- ✅ Create custom detection rules
- ✅ Investigate security incidents
- ✅ Respond to threats automatically
- ✅ Build compliance dashboards

### SIEM Expertise
- ✅ Understand SIEM architecture (manager, indexer, dashboard)
- ✅ Write detection rules in Wazuh syntax
- ✅ Use MITRE ATT&CK for mapping
- ✅ Configure alerts and automated responses
- ✅ Build custom dashboards and reports
- ✅ Perform threat hunting

---

## 📊 IP Address Reference

Keep this handy:

```
Network:                     192.168.100.0/24
Gateway:                     192.168.100.1
Wazuh Server:                192.168.100.10
Windows Victim:              192.168.100.20
Kali Attacker:               192.168.100.30

Access Wazuh Dashboard from host PC: https://192.168.100.10
Default username: admin
```

---

## 🔐 Security Notes

### Lab vs. Production
- **This lab is intentionally insecure** (firewall off, weak passwords, etc.)
- **Never apply these settings to production systems**
- **Use this lab only for learning in isolated environments**

### What NOT to Do
- ❌ Run this lab on a production network
- ❌ Use weak passwords in real systems
- ❌ Disable firewalls on production machines
- ❌ Leave systems accessible to the internet
- ❌ Share agent credentials publicly

### Best Practices Demonstrated
- ✅ Network isolation (Host-Only network)
- ✅ Centralized logging (SIEM)
- ✅ Real-time alerting (email, webhook)
- ✅ Automated responses (blocking, isolation)
- ✅ Continuous monitoring (agents)

---

## 📚 Further Learning Resources

### Official Documentation
- Wazuh Docs: https://documentation.wazuh.com/
- MITRE ATT&CK: https://attack.mitre.org/
- Kali Tools: https://tools.kali.org/

### Certifications to Pursue
1. **CompTIA Security+** (entry-level)
2. **CEH** (Certified Ethical Hacker)
3. **GIAC GCIA** (Certified Intrusion Analyst)
4. **eLearnSecurity eCDFP** (Defensive Security)

### Hands-On Practice Platforms
- HackTheBox: https://www.hackthebox.com/
- TryHackMe: https://www.tryhackme.com/
- Cybrary: https://www.cybrary.it/
- eJPT (eJPTv2): https://ejpt.ine.com/

### YouTube Channels
- Wazuh Official: https://www.youtube.com/c/Wazuh
- Cybersecurity Tutorials
- SOC Analysis walkthroughs

---

## 📝 How to Use These Documents

### Format
Each markdown file follows this structure:
1. **Overview** - What you'll build in this phase
2. **Prerequisites** - What you need before starting
3. **Step-by-Step Instructions** - Detailed walkthrough
4. **Code/Commands** - Copy-paste ready commands
5. **Troubleshooting** - Common issues and fixes
6. **Verification Checklist** - Confirm completion

### Tips for Success
1. **Follow each phase in order** - Later phases depend on earlier ones
2. **Don't skip steps** - Even "optional" steps provide value
3. **Take your time** - This is a learning exercise, not a race
4. **Document your progress** - Write down IPs, passwords, discoveries
5. **Test everything** - Verify each step works before moving on
6. **Ask for help** - Wazuh community is very helpful

### Setting Up Your Learning Environment
1. **Create a folder** for all documentation
2. **Print or save PDFs** of all markdown files
3. **Create a lab journal** documenting your discoveries
4. **Save credentials** in a safe location
5. **Take screenshots** of successful attacks and detections

---

## 🎯 Success Criteria

By the end of this project, you should be able to:

### Level 1 (Phase 1-3): Setup
- [ ] Create and configure virtual networks
- [ ] Deploy SIEM and endpoints
- [ ] Verify agent connectivity

### Level 2 (Phase 4-5): Attack & Detect
- [ ] Launch various attacks
- [ ] Observe detections in real-time
- [ ] Understand attack signatures

### Level 3 (Phase 6): Advanced
- [ ] Create custom detection rules
- [ ] Investigate incidents thoroughly
- [ ] Set up automated responses
- [ ] Build compliance dashboards
- [ ] Explain attacks to non-technical people

---

## 🚨 Important Notes

### Before You Start
1. Ensure you have **all required ISOs** downloaded (Ubuntu, Windows 10, Kali)
2. **Backup important data** on your host PC
3. Ensure **VMware is properly installed** and licensed
4. Verify **network connectivity** on your host PC
5. Set aside **uninterrupted time blocks** for building VMs

### During the Project
1. **Take notes** on what you learn
2. **Document errors** you encounter and how you fixed them
3. **Save credentials** securely (consider password manager)
4. **Screenshot successful milestones** for portfolio
5. **Join communities** (Wazuh Slack, Reddit r/cybersecurity)

### After Completion
1. **Write a blog post** about your experience
2. **Create a portfolio project** showcasing your lab
3. **Practice similar scenarios** on HackTheBox/TryHackMe
4. **Consider adding more VMs** (domain controller, web server)
5. **Pursue certifications** using this lab as foundation

---

## 📞 Getting Help

### If You Get Stuck

1. **Check the troubleshooting section** of the relevant phase
2. **Re-read the step** carefully (often you'll spot your own mistake)
3. **Search online** for the specific error message
4. **Check Wazuh official docs** at https://documentation.wazuh.com/
5. **Join Wazuh Slack** community for real-time help
6. **Post on relevant subreddits** (r/cybersecurity, r/Wazuh)

### Useful Commands for Debugging

```bash
# Check service status
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-agent

# View logs
sudo tail -50 /var/ossec/logs/ossec.log

# Check connectivity
ping 192.168.100.10

# Verify IP config
hostname -I
ipconfig /all  # Windows

# Restart services
sudo systemctl restart wazuh-manager
Restart-Service WazuhSvc  # Windows PowerShell
```

---

## 🎓 Certification Preparation

### CompTIA Security+
**Relevant topics covered**:
- Threat detection and monitoring
- SIEM fundamentals
- Incident response procedures
- Log analysis
- Risk management

### CEH (Certified Ethical Hacker)
**Relevant topics covered**:
- Network scanning and enumeration
- Exploitation techniques
- Post-exploitation
- Covering tracks

### GIAC GCIA (Certified Intrusion Analyst)
**Relevant topics covered**:
- Network traffic analysis
- Intrusion detection
- Log analysis
- Forensics

---

## 📊 Project Timeline

```
Day 1-2:  Phase 1 (Network Setup)
Day 3-4:  Phase 2 (Wazuh Server)
Day 5-6:  Phase 3 (Windows Victim)
Day 7:    Phase 4 (Kali Attacker)
Day 8-10: Phase 5 (Attack & Detect)
Day 11-14: Phase 6 (Analysis & Rules)

Total: 14 days (can be condensed to 8-10 with full-day sessions)
```

---

## ✅ Final Checklist Before Starting

- [ ] VMware Workstation installed and working
- [ ] All required ISOs downloaded
- [ ] At least 300GB free disk space
- [ ] 24GB RAM available on host
- [ ] Good internet connection (for downloads)
- [ ] 2-3 weeks of dedicated learning time
- [ ] Quiet workspace for focused work
- [ ] Note-taking materials ready
- [ ] Password manager set up
- [ ] Backup of important data completed

---

## 🎉 Congratulations!

You now have a complete, step-by-step guide to build a production-grade SOC lab. This isn't just theoretical knowledge — you'll have **hands-on experience** with:
- Real attack simulations
- Real detection systems
- Real incident response
- Real SIEM configuration

**This lab will be valuable for:**
- Job interviews ("Tell me about your SOC experience")
- Certification exams (hands-on foundation)
- Career portfolio (proof of expertise)
- Continuous learning (platform for experimentation)

---

## 📖 Document Version

**Version**: 1.0
**Created**: January 2024
**Based on**: Wazuh 4.14, VMware Workstation, Ubuntu 22.04 LTS, Windows 10, Kali Linux
**Tested on**: Ryzen 5 hexacore, 24GB RAM, 500GB SSD

---

## 🔗 Next Steps

1. **Start with Phase 1** - Set up your network
2. **Follow each phase in order**
3. **Complete verification checklist** after each phase
4. **Document your journey**
5. **Share your experience** in SOC communities
6. **Build on this lab** with more VMs and scenarios

---

**Good luck, and welcome to the world of cybersecurity! 🛡️**

*Remember: The best way to understand security is to experience both attacking AND defending. This lab gives you both perspectives.*
