# 🛡️ Blue Team Home Lab — SSH Brute-Force Detection

A cybersecurity home lab simulating a real SSH brute-force attack and its detection using open-source tools.

---

## 🏗️ Lab Architecture

```
┌─────────────────────────────────────────┐
│              VMware Workstation          │
│                                          │
│  ┌─────────────┐    ┌─────────────────┐ │
│  │  Kali Linux │───▶│  Ubuntu Server  │ │
│  │  (Attacker) │    │    (Target)     │ │
│  └─────────────┘    └─────────────────┘ │
│              NAT Network                 │
└─────────────────────────────────────────┘
```

---

## 🎯 Project Goals

- Simulate a realistic SSH brute-force attack in a controlled environment
- Capture and analyze network traffic with Wireshark
- Detect and block the attack automatically with Fail2ban
- Document the full attack lifecycle from recon to detection

---

## 🧰 Tools Used

| Tool | Role |
|------|------|
| Nmap | Network reconnaissance |
| Hydra | SSH brute-force attack |
| Wireshark | Traffic capture & analysis |
| Fail2ban | Intrusion detection & blocking |
| VMware Workstation | Lab virtualization |

---

## 📋 Phases

### Phase 1 — Lab Setup
- Configured two isolated VMs on a shared network
- Validated bidirectional connectivity between attacker and target

### Phase 2 — Reconnaissance
- Ran Nmap scan to discover open ports and running services

```bash
nmap -sV 192.168.x.130
```
<img width="631" height="247" alt="image" src="https://github.com/user-attachments/assets/19dd3d90-6de5-4601-b57a-8247acad401f" />

**Result:** SSH (22), HTTP (80), FTP (21) identified on target.

### Phase 3 — SSH Brute-Force Attack
- Created a weak test account (`testuser / password123`)
- Launched dictionary attack using Hydra with the rockyou.txt wordlist

```bash
hydra -l testuser -P /usr/share/wordlists/rockyou.txt ssh://TARGET_IP -t 4 -V
```
<img width="633" height="267" alt="image" src="https://github.com/user-attachments/assets/c8389d8b-0fb5-4a74-aebc-0f0168d8bc57" />

**Result:** Password cracked after ~18 minutes.

### Phase 4 — Traffic Analysis
- Captured live attack traffic with Wireshark
- Identified brute-force signature: repeated TCP SYN → handshake → FIN cycles on port 22
- Exported capture as `brute_force_ssh.pcap`
<img width="1718" height="841" alt="Capture d&#39;écran 2026-04-03 232345" src="https://github.com/user-attachments/assets/7f2c7e37-aab9-4d29-b2aa-e725952fef4e" />


### Phase 5 — Detection & Blocking
- Configured Fail2ban to monitor SSH authentication logs
- Policy: ban after 5 failed attempts within 10 minutes

```ini
[sshd]
enabled = true
maxretry = 5
bantime = 600
findtime = 600
```
<img width="448" height="164" alt="Capture d&#39;écran 2026-04-03 234812" src="https://github.com/user-attachments/assets/543bda0f-9d92-4558-8b40-5455ebbd49a6" />

**Result:** Attacker IP auto-banned after 20 failed attempts. Attack terminated.

---

## 📁 Repository Contents

```
├── brute_force_ssh.pcap     # Wireshark capture of the attack
├── screenshots/
│   ├── nmap_scan.png
│   ├── hydra_attack.png
│   ├── wireshark_capture.png
│   └── fail2ban_ban.png
└── README.md
```

---

## 🔑 Key Takeaways

- **Weak passwords are easily cracked** — `password123` appears early in public wordlists
- **Brute-force attacks have a clear network signature** — visible in Wireshark as repeated connection cycles on port 22
- **Automated detection works** — Fail2ban blocked the attack without manual intervention
- **Defense-in-depth matters** — combining log monitoring + firewall banning is more robust than either alone

---
