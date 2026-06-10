# 🎯 LLMNR/NBT-NS Poisoning - CTF Challenge

## 📋 Overview

**LLMNR/NBT-NS Poisoning** is an interactive, browser-based Capture The Flag (CTF) challenge designed for cybersecurity training. This challenge focuses on detecting Link-Local Multicast Name Resolution (LLMNR) and NetBIOS Name Service (NBT-NS) poisoning attacks where attackers use tools like Responder to intercept name resolution requests and capture NTLMv2 hashes. Participants analyze packet captures to identify spoofed responses and extract compromised credentials.

## 🎯 Learning Objectives

By completing this CTF, participants will learn:

- **Attack Recognition**: Identify LLMNR/NBT-NS poisoning techniques in network traffic
- **Packet Analysis**: Analyze captured packets for spoofed name resolution responses
- **Hash Extraction**: Extract NTLMv2 hashes from captured authentication attempts
- **GPO Remediation**: Disable LLMNR and NBT-NS via Group Policy
- **Monitoring**: Set up detection for similar poisoning attacks

## 🛠️ Challenge Tasks (5 Total)

| Task | Description | Skill Focus |
|------|-------------|-------------|
| **Task 1** | Identify the attack technique (LLMNR poisoning) | Attack Recognition |
| **Task 2** | Find spoofed LLMNR responses (attacker IP) | Packet Analysis |
| **Task 3** | Extract NTLMv2 hash and username | Hash Extraction |
| **Task 4** | Mitigation via GPO (disable multicast name resolution) | Remediation |
| **Task 5** | Monitor for similar events (Event 4624) | Detection Engineering |

## 🚀 Quick Start

### Prerequisites
- A modern web browser (Chrome, Firefox, Edge, Safari)
- No server required - runs entirely in the browser
- No installation needed

### Access the Challenge
1. Open the HTML file directly in your browser
2. Enter your name
3. Use the password: `45_2026`
4. Complete all 5 tasks to capture the flag

### Hosting on GitHub Pages
1. Fork or clone this repository
2. Go to repository Settings > Pages
3. Select the branch (usually `main`) and save
4. Access via `https://your-username.github.io/repository-name`

## 🎮 How to Play

### Login
```
Password: 45_2026
Name: Enter any name (progress is saved locally)
```

### Game Features

- **Attack Flow Diagram**: Visual representation of LLMNR poisoning attack with pulsing attacker node
- **Packet Capture Snippet**: Simulated network packets with syntax highlighting
- **Highlighted Fields**: Red for spoofed/malicious data, yellow for suspicious queries
- **NTLMv2 Hash Display**: Extracted username and hash values from captured traffic
- **GPO Reference**: Group Policy path for disabling LLMNR
- **Event ID Reference**: Windows Event IDs for monitoring NTLM authentication
- **Answer Validation**: Immediate feedback on submitted answers
- **Progress Tracking**: Local storage saves your progress across sessions

### Completing Tasks
1. Read each task description carefully
2. Analyze the attack flow diagram and packet capture
3. Identify spoofed responses and extracted credentials
4. Type your answer in the input field
5. Click "Submit" to validate
6. Complete all 5 tasks to reveal the flag

## 🏆 Flag

```
FLAG{LLMNR_POISONING}
```

The flag is revealed only after completing all 5 tasks successfully.

## 📊 Challenge Details

### Attack Flow Diagram

```
⚔️ LLMNR Poisoning Attack Flow:
├── Victim (192.168.1.100)
│   ├── ❶ LLMNR Query for \\fileshare
│   ├── ❷ Spoofed Response from attacker
│   └── ❸ NTLMv2 Hash Sent to attacker
│
└── Attacker (10.0.1.50)
    ├── Listens for LLMNR/NBT-NS queries
    ├── Responds to all queries with own IP
    └── Captures NTLMv2 hashes for offline cracking
```

### Packet Capture Snippet

```
[LLMNR Query] 192.168.1.100 → 224.0.0.252 (LLMNR Multicast)
  Query: \\fileshare (non-existent host)
  Type: A/AAAA

[LLMNR Response] 10.0.1.50 → 192.168.1.100 ⚠️ SPOOFED!
  Response: \\fileshare = 10.0.1.50
  TTL: 30

[SMB Connection] 192.168.1.100 → 10.0.1.50
  Negotiate Protocol Request
  NTLMSSP_NEGOTIATE

[NTLM Challenge] 10.0.1.50 → 192.168.1.100
  NTLMSSP_CHALLENGE

[NTLM Auth] 192.168.1.100 → 10.0.1.50
  NTLMv2 Hash Captured!
  Username: jdoe
  Domain: CORP
  Hash: jdoe::CORP:1122334455667788:AAAABBBBCCCCDDDD...

[LLMNR Query] 192.168.1.101 → 224.0.0.252
  Query: \\printserver

[LLMNR Response] 10.0.1.50 → 192.168.1.101 ⚠️ SPOOFED!
  Response: \\printserver = 10.0.1.50

[SMB Connection] 192.168.1.101 → 10.0.1.50
  NTLMv2 Hash Captured!
  Username: asmith
  Domain: CORP
```

## 🔍 Investigation Walkthrough

### Task 1: Identify Attack Technique
This is **LLMNR poisoning**. The attack works because:
- When DNS resolution fails, Windows falls back to LLMNR
- LLMNR uses multicast to query all hosts on the network
- Any host can respond to LLMNR queries
- Attackers respond to all queries claiming to be the requested host
- Victims then authenticate to the attacker's system
- NTLMv2 hashes are captured during this authentication

### Task 2: Find Spoofed Responses
The attacker IP is **10.0.1.50**. This is identified by:
- Consistently responding to LLMNR queries for non-existent hosts
- Responding with its own IP address (spoofing)
- Responses coming milliseconds after queries
- Responses targeting multiple victim IPs
- No legitimate service should respond to all name queries

### Task 3: Extract NTLMv2 Hash
The compromised username is **jdoe**. The hash was captured when:
- User tried to access a non-existent share (\\fileshare)
- LLMNR query went out looking for the host
- Attacker responded claiming to be \\fileshare
- Victim's system automatically attempted SMB authentication
- NTLMv2 hash was sent to the attacker during authentication
- The hash can be cracked offline to reveal the password

### Task 4: Mitigation via GPO
The GPO setting is **"Turn off multicast name resolution"**. This:
- Disables LLMNR on all domain-joined systems
- Prevents Windows from sending multicast queries
- Eliminates the attack vector for LLMNR poisoning
- Located in Computer Configuration → Administrative Templates → Network → DNS Client
- Should be combined with disabling NBT-NS for complete protection

### Task 5: Monitor for Similar Events
Monitor **Event ID 4624** (Logon events) for:
- NTLM authentication instead of Kerberos
- Logons from unexpected IP addresses
- Multiple failed NTLM attempts followed by success
- Service account logons from workstations
- Also monitor UDP port 5355 (LLMNR) traffic patterns

## 🎨 Visual Features

- **Attack Flow Diagram**: Visual representation with pulsing red attacker node
- **Directional Arrows**: Attack flow arrows showing query/response/hash capture
- **Color-coded Packet Fields**: Red for spoofed attacker data, yellow for suspicious queries
- **Packet Snippet Viewer**: Monospace display with syntax highlighting
- **Numbered Attack Steps**: Sequential flow markers (❶, ❷, ❸)
- **Progress Indicators**: Visual completion status for each task
- **Glowing Flag Animation**: Celebratory golden flag reveal
- **Toast Notifications**: Non-intrusive success/error messages
- **Dark Theme**: Purple-accented UI for network attack theme

## 💾 Data Storage

- **Progress**: Saved in browser's `localStorage`
- **Persistence**: Progress survives page refreshes
- **Privacy**: All data stays on the user's device
- **Reset**: Clear browser data to reset progress

## 🛡️ LLMNR/NBT-NS Poisoning Detection

### High-Fidelity Indicators:
- Multiple LLMNR queries for non-existent hosts
- Single IP responding to diverse LLMNR queries
- NTLM authentication to unexpected IP addresses
- Failed DNS queries immediately followed by LLMNR traffic
- SMB connections to non-domain hosts

### Medium-Fidelity Indicators:
- LLMNR traffic during non-business hours
- NTLMv2 instead of Kerberos authentication
- Multiple failed name resolution attempts
- Unusual network traffic on UDP 5355

### Network Signatures:
```
Protocol: LLMNR (UDP 5355)
Direction: Multicast (224.0.0.252)
Query Types: A, AAAA
Response: Unicast to querying host
Attack Pattern: Any response to any query
```

## 📁 File Structure

```
llmnr-nbtns-poisoning/
│
├── index.html          # Main CTF challenge file
├── README.md           # This documentation
└── (no other files required)
```

## 🔧 Technical Implementation

- **Pure Frontend**: HTML5, CSS3, JavaScript (Vanilla)
- **No Dependencies**: Zero external libraries
- **Responsive Design**: Works on desktop and mobile
- **Animations**: CSS keyframe animations for attacker node
- **Storage**: Browser localStorage API
- **Gamification**: Progress tracking, badge system, visual rewards
- **Packet Visualization**: Highlighted packet capture display

## 📊 LLMNR vs NBT-NS Comparison

| Feature | LLMNR | NBT-NS |
|---------|-------|--------|
| Protocol | UDP 5355 | UDP 137 |
| Scope | Link-local multicast | Local network broadcast |
| Address | 224.0.0.252 (IPv4) | 255.255.255.255 |
| Modern OS | Windows Vista+ | Legacy Windows |
| Disable GPO | Turn off multicast name resolution | NetBIOS over TCP/IP |
| Attack Tool | Responder, Inveigh | Responder, Inveigh |

## 🎓 Educational Use Cases

- **Cybersecurity Training Programs**
- **SOC Analyst Onboarding**
- **Network Security Workshops**
- **Blue Team Exercises**
- **Incident Response Training**
- **Academic Courses** (Network Security, Windows Security)
- **Self-paced Learning**
- **Purple Team Exercises**

## 🔄 Version History

- **v1.0** - Initial release
  - 5 tasks with validation
  - Attack flow diagram with pulsing attacker node
  - Simulated packet capture with highlighted fields
  - NTLMv2 hash extraction display
  - GPO remediation reference
  - Local storage progress tracking
  - Student login system

## 👥 Target Audience

- Security Operations Center (SOC) Analysts
- Incident Response Team Members
- Network Security Engineers
- Threat Hunters
- Windows System Administrators
- Cybersecurity Students
- IT Security Professionals
- Blue Team Practitioners

---

**Happy Poisoning Detection! 🎯**
