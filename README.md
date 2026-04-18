# 🏢 Enterprise Homelab SIEM Environment

A fully functional enterprise-style Security Operations Center (SOC) environment built from scratch on a single physical machine using nested virtualization. This project documents the complete build lifecycle — architecture decisions, configuration steps, troubleshooting, and the final working SIEM pipeline.

---

## 🎯 Project Goal

To simulate a real enterprise network environment for hands-on SOC analyst training, detection engineering practice, and attack simulation — all running on one Windows 11 host machine.

---

## 🗺️ Architecture Overview

```
Windows 11 Host
└── Hyper-V (Type 1 Hypervisor)
    └── Proxmox VE (Nested Hypervisor)
        ├── pfSense Firewall (VLAN segmentation + firewall rules)
        ├── Windows Server — DC01 (Active Directory, DNS, lab.local)
        ├── Windows Workstation (Domain-joined endpoint)
        ├── Kali Linux (Attack machine — isolated ATTACK VLAN)
        └── Splunk Enterprise (SIEM — log collection and analysis)
```

---

## 🔀 VLAN Segmentation

| VLAN | Purpose | Notes |
|---|---|---|
| SERVER VLAN | Domain controller, Splunk | Primary infrastructure |
| WORKSTATION VLAN | User endpoint simulation | Generates realistic user logs |
| ATTACK VLAN | Kali Linux attacker machine | Isolated — no direct server access |

Firewall rules enforce traffic flow between VLANs. TCP port 9997 is explicitly allowed from SERVER VLAN to Splunk for Universal Forwarder communication.

---

## 📡 SIEM Pipeline

```
DC01 (Windows Server)         ──► Universal Forwarder ──► Splunk Enterprise
Windows Workstation            ──► Universal Forwarder ──► Splunk Enterprise
```

**Log sources ingested:**
- Windows Security Event Log
- Windows System Event Log
- Windows Application Event Log

**Splunk receiving port:** TCP 9997

---

## ⚙️ Key Configuration Steps

### 1. Enable Nested Virtualization in Hyper-V
```powershell
Set-VMProcessor -VMName "ProxMoxServer" -ExposeVirtualizationExtensions $true
```

### 2. Enable Hyper-V
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

### 3. Verify Splunk Receiving Port
```powershell
netstat -ano | findstr 9997
```

### 4. Universal Forwarder — outputs.conf
```ini
[tcpout]
defaultGroup = default-autolb-group

[tcpout:default-autolb-group]
server = 192.168.20.10:9997
```

### 5. Universal Forwarder — inputs.conf
```ini
[WinEventLog://Security]
disabled = 0

[WinEventLog://System]
disabled = 0

[WinEventLog://Application]
disabled = 0
```

### 6. Restart Universal Forwarder
```powershell
Restart-Service splunkforwarder
```

### 7. Validate Forwarder Input Status
```powershell
splunk list inputstatus
```

---

## 🔧 Troubleshooting Encountered

| Issue | Root Cause | Fix |
|---|---|---|
| Nested VMs getting APIPA addresses | Hyper-V MAC filtering blocking DHCP | Enable MAC spoofing on Proxmox VM NIC |
| Proxmox not detecting virtualization | Extensions not exposed by default | Run Set-VMProcessor command |
| Splunk not receiving logs | Firewall rule blocking TCP 9997 | Add explicit allow rule in pfSense |
| Universal Forwarder not loading inputs | Missing inputs.conf | Create inputs.conf and restart service |

---

## ✅ Final State

- Hyper-V successfully hosting Proxmox via nested virtualization
- Proxmox hosting all five VMs with stable networking
- pfSense enforcing VLAN segmentation and firewall rules
- Active Directory domain (lab.local) fully operational
- Windows Workstation joined to domain
- Splunk Enterprise receiving Security, System, and Application logs from two endpoints
- Kali Linux isolated on ATTACK VLAN and ready for attack simulation

---

## 🛠️ Skills Demonstrated

- Nested virtualization configuration (Hyper-V + Proxmox)
- Enterprise network segmentation with VLANs and firewall rules
- Active Directory deployment and domain configuration
- Splunk Enterprise SIEM installation and log ingestion
- Universal Forwarder deployment and configuration
- Network troubleshooting across virtualization layers
- SOC environment design and documentation

---

## 🔮 Planned Enhancements

- [ ] Add Sysmon for deeper endpoint telemetry
- [ ] Build Splunk detection dashboards
- [ ] Implement Windows Event Forwarding (WEF)
- [ ] Add Suricata or Zeek for network-level monitoring
- [ ] Document full attack simulation from Kali and trace through Splunk

---

## 👨‍💻 Author

**Christopher O'Keefe**
IT Professional | Cybersecurity BAS | CompTIA A+ | ISC2 CC
[LinkedIn](https://linkedin.com/in/christopherokeefe93) | [GitHub](https://github.com/TheCaptainCJ)

> ⚠️ This environment is for educational and authorized use only.
