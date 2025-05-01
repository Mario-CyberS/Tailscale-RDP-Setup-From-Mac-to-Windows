# Tailscale RDP Setup From Mac To Windows  
This project documents the configuration and secure setup of **Remote Desktop Protocol (RDP)** access from a **MacBook** to a **Windows PC** using **Tailscale**, enabling private, encrypted access with no need for port-forwarding or static IPs.

---

## 🎯 Objective  
To establish a secure, zero-trust private connection from macOS to a Windows machine via RDP, using **Tailscale IP routing**, **ACLs**, and **MagicDNS** for seamless access.

---

## 🔍 Why Use Tailscale for RDP?  
Tailscale builds a private mesh network using WireGuard, ideal for:
- Encrypted peer-to-peer connectivity between devices  
- No public IP or firewall port forwarding required  
- Easy device management with ACLs and tags  
- Secure RDP access from any OS, even over the internet

---

## 📚 Skills Learned  
- Deploying and registering devices in a private tailnet  
- Using ACLs and tags for secure access control  
- Troubleshooting RDP access and firewall settings  
- Verifying peer-to-peer routing and DNS resolution  
- Configuring Microsoft Remote Desktop on macOS

---

## 🛠️ Tools Used  
<div>
  <a href="https://tailscale.com/" target="_blank"><img src="https://img.shields.io/badge/-Tailscale-005AE0?&style=for-the-badge&logo=Tailscale&logoColor=white" />
  <img src="https://img.shields.io/badge/-Microsoft_RDP-0078D4?&style=for-the-badge&logo=Windows&logoColor=white" />
  <img src="https://img.shields.io/badge/-macOS-000000?&style=for-the-badge&logo=Apple&logoColor=white" />
</div>

---

## 📝 Deployment Steps

### 1. Install Tailscale on Both Devices

#### On Windows PC:
- Installed from [tailscale.com/download](https://tailscale.com/download)
- I logged in with GitHub, but go ahead an create an account if you please.
- Device name: `PC`
- Assigned Tailscale IP: <IP>

#### On MacBook:
- Installed Tailscale app
- Joined same tailnet with the same GitHub/account login

---

### 2. Configure ACLs and Tags
Go to your TailScale admin Web interface and go to the Access Controls.
ACL Configuration:
```json
{
  "acls": [
    {
      "action": "accept",
      "src": ["Mario-CyberS@github"],
      "dst": ["tag:home:3389"]
    }
  ],
  "tagOwners": {
    "tag:home": ["Mario-CyberS@github"]
  }
}
```
Tagged the Windows PC as tag:home, enabling full access for your user.
Go to https://login.tailscale.com/admin/machines in the machines tab, you will see your PC and Macbook, click the 3 dots on PC machine and add the “home” tag.

### 3. RDP Configuration on Mac
Used Microsoft Remote Desktop with:
In the Microsoft Remote Desktop app:

Field	Value
PC Name:	100.94.168.104 or homelab (DNS name)
Credentials:	homelab\owner and its pass
Friendly Name:	Home Windows
Gateway:	(None)
Reconnect:	Enabled

### 4. Fix RDP Error 0x204
Initial attempt returned:
Unable to connect – Error code: 0x204
This is due to the Windows Fire wall, run the command below

Fixes Applied:

Resolved by:
Enabling RDP:
Settings > System > Remote Desktop → ON

Allowing RDP in Windows Firewall:
```bash
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

Verifying Tailscale IP with:
```bash
tailscale ip -4
```

### 5. Test DNS and Connectivity from Mac
nslookup PC.tail******.ts.net
ping PC.tail******.ts.net
or
ping <PC-TailScal-IP>

### 6. Final Security Enhancements
Feature	Status
ACLs with tag restrictions: Enabled
Identity restriction: GitHub only (Mario-CyberS)
NAT traversal (P2P): Enabled
Firewall for RDP: Enabled with PowerShell
MagicDNS: Enabled via device config

### 7. Make the Firewall Rule Persistent and Force it ON at Boot
You can create a Startup Task that automatically runs Enable-NetFirewallRule each time your PC boots up.

Steps:
- Open Task Scheduler
- Click Create Task (not "Basic Task")
- Under General tab:
Name: Enable RDP Firewall
Run whether user is logged on or not
Run with highest privileges

- Under Triggers:
New → At Startup

- Under Actions:
Start a program →
Program/script: 
```bash
powershell.exe 
```
Arguments:
```bash
-Command "Enable-NetFirewallRule -DisplayGroup 'Remote Desktop'"
```
- Save and exit
Now every boot will re-enable the firewall rules.

Security Impact:
This is safe because you’re only re-enabling the Windows built-in RDP rules — you're not creating new loose ports.
However, always ensure you're connecting through Tailscale IP only, not allowing public internet access.

---

### 👨‍💻 Author
Mario Tagaras | Florida State University Alum













