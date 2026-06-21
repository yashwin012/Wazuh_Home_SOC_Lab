# Phase 1: VMware Workstation Host-Only Network Setup

## Overview
This phase sets up an **isolated network** for your SOC lab. All three VMs (Wazuh Server, Windows Victim, Kali Attacker) will communicate on this network without reaching your host machine or the internet. This isolation is critical for safe attack simulation.

## Why Host-Only Network?
- **Sandboxed environment**: No attacks leak to your real network
- **Full control**: You control which VM can talk to which
- **Realistic lab**: Simulates a corporate air-gapped security lab
- **Safe to experiment**: Zero risk of harming your host or network

---

## Step-by-Step Instructions

### Step 1: Open VMware Workstation Virtual Network Editor

1. Launch **VMware Workstation Pro/Player**
2. Click on **Edit** menu (top menu bar)
3. Select **Virtual Network Editor**

You should see a window with existing virtual networks (usually VMnet0, VMnet1, VMnet8).

### Step 2: Create a New Host-Only Network

1. Click the **Add Network...** button
2. A dialog box appears
3. Select **VMnet2** from the dropdown (or the next available VMnet)
   - If VMnet2 is already in use, choose VMnet3, VMnet4, etc.

### Step 3: Configure the Network

In the Virtual Network Editor window for your selected VMnet:

#### Network Type
- Make sure **Host-only** is selected (radio button)
- Do NOT select "NAT" or "Bridged"

#### Network Configuration
Look for the section that shows:
- **Subnet IP**: Change this to `192.168.100.0`
- **Subnet Mask**: Set to `255.255.255.0`

#### DHCP Settings (Important!)
- **Uncheck** the box labeled "Use DHCP to distribute IP address ranges to VMs"
- We will assign **static IPs manually** inside each VM
- This gives you precise control over which VM has which IP

#### Gateway
- Leave the gateway field as auto-generated or set to `192.168.100.1`

### Step 4: Apply Changes

1. Click **OK** or **Apply** button
2. Click **OK** on the Virtual Network Editor confirmation dialog
3. VMware will apply the network settings (takes a few seconds)

### Step 5: Verify the Network Exists

1. Go back to **Edit → Virtual Network Editor**
2. Check that your new network (VMnet2/VMnet3) appears in the list
3. Verify it shows:
   - **Type**: Host-only
   - **Subnet**: 192.168.100.0
   - **Mask**: 255.255.255.0
   - **DHCP**: Disabled

---

## Network IP Allocation Schema

Your lab will use these IPs:

| Component | IP Address | Purpose |
|-----------|-----------|---------|
| Gateway (VMware) | 192.168.100.1 | Network gateway (auto) |
| Wazuh Server | 192.168.100.10 | Central SOC, logs aggregation |
| Windows Victim | 192.168.100.20 | Target for attacks, monitored |
| Kali Attacker | 192.168.100.30 | Red team, launches attacks |
| Available | 192.168.100.31-254 | Future use |

**Key Point**: You'll manually set these IPs inside each VM's operating system, not relying on VMware's DHCP.

---

## Network Diagram (Reference)

```
Host PC (Your Computer)
│
└─ VMware Workstation
   │
   └─ VMnet2 (Host-Only, 192.168.100.0/24)
      │
      ├─ Wazuh Server (192.168.100.10)
      │  └─ Receives logs from agents
      │
      ├─ Windows 10 Victim (192.168.100.20)
      │  └─ Runs Wazuh Agent, sends logs
      │
      └─ Kali Attacker (192.168.100.30)
         └─ Launches attacks on victim
```

No traffic escapes this network. Your host PC cannot directly access 192.168.100.x (it's isolated), but you **can access the Wazuh dashboard via localhost port forwarding or the VM's IP if you enable it**.

---

## Troubleshooting

### Issue: Virtual Network Editor button is grayed out
**Solution**: You may need to run VMware as Administrator. Close VMware, right-click it, select "Run as Administrator", then retry.

### Issue: Cannot see VMnet2 option
**Solution**: 
1. Click "Add Network" again
2. Manually type "VMnet2" in the VMnet field
3. Select Host-only type
4. Click OK

### Issue: "Subnet IP already in use"
**Solution**: Choose a different subnet like `192.168.101.0` or `192.168.50.0`. Just remember to update all subsequent IP assignments in the later phases.

### Issue: DHCP option cannot be unchecked
**Solution**: This is normal on some VMware versions. Just leave it unchecked when creating your VMs (don't use DHCP). You'll assign static IPs manually inside each VM's OS.

---

## What Happens Next

After completing this phase:
1. Your VMnet2 network is ready
2. When you create VMs, you'll select this network as their adapter
3. Each VM will get a static IP in the 192.168.100.x range
4. All three VMs can communicate with each other
5. Your real network remains isolated and safe

---

## Verification Checklist

- [ ] Virtual Network Editor opened successfully
- [ ] Created a new Host-Only virtual network (VMnet2 or VMnetX)
- [ ] Subnet IP set to 192.168.100.0
- [ ] Subnet Mask set to 255.255.255.0
- [ ] DHCP is disabled on this network
- [ ] Changes applied and confirmed in VMware settings
- [ ] Network appears in Virtual Network Editor list
- [ ] IP allocation schema documented (use table above as reference)
- [ ] Understand the network is completely isolated from your host
- [ ] Ready to create the first VM (Wazuh Server)

---

## Next Phase
Proceed to **Phase 2: Wazuh Server VM Setup** once all checklist items are complete.
