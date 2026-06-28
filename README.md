# Snort 3 Custom Detection Rules Lab

> A hands-on cybersecurity project demonstrating the development, testing, and validation of custom **Snort 3 Intrusion Detection System (IDS)** rules in an isolated virtual environment.

---

# Overview

This project demonstrates how to build and validate custom Snort 3 detection rules against common cyber attacks. An isolated lab was created using three virtual machines connected through an internal VirtualBox network.

The objective was to understand how network-based intrusion detection systems detect malicious activities, develop custom signatures, generate attack traffic, and validate detections using Wireshark.

---

# Lab Architecture

The lab consists of three virtual machines connected to an isolated **192.168.10.0/24** network.

| Machine        | IP Address        | Role                             |
| -------------- | ----------------- | -------------------------------- |
| Kali Linux     | **192.168.10.15** | Attacker                         |
| Kali Linux     | **192.168.10.10** | Monitoring (Snort 3 + Wireshark) |
| Ubuntu Desktop | **192.168.10.5**  | Victim                           |

The monitoring VM runs Snort 3 and Wireshark to inspect all traffic exchanged between the attacker and the victim.

---

# Services Running on the Victim

* Apache2 (HTTP)
* OpenSSH
* DNS Server
* FTP Server (vsftpd)
* ICMP (Ping)

---

# Detection Rules

The following custom detection rules were implemented.

| SID     | Detection                                |
| ------- | ---------------------------------------- |
| 1000001 | SSH Brute Force                          |
| 1000002 | SYN Flood                                |
| 1000003 | High Frequency DNS Queries               |
| 1000004 | Large DNS Query (Possible DNS Tunneling) |
| 1000005 | ICMP Exfiltration                        |
| 1000006 | TCP SYN Port Scan                        |
| 1000007 | TCP NULL Scan                            |
| 1000008 | TCP XMAS Scan                            |
| 1000009 | Outbound FTP Connection                  |
| 1000010 | SQL Injection                            |
| 1000011 | Command Injection                        |
| 1000012 | Access to `/etc/passwd`                  |
| 1000013 | Suspicious `sqlmap` User-Agent           |

---

# Repository Structure

```text
Snort3-Custom-Detection-Rules/
│
├── README.md
├── LICENSE
│
├── config/
│   └── snort.lua
│
├── rules/
│   └── local.rules
│
├── docs/
│   ├── lab_architecture.png
│   └── test_commands.md
│
├── screenshots/
│   ├── ssh_bruteforce/
│   ├── syn_flood/
│   ├── dns_high_frequency/
│   ├── dns_tunneling/
│   ├── icmp_exfiltration/
│   ├── ftp_exfiltration/
│   ├── sql_injection/
│   ├── command_injection/
│   ├── passwd_access/
│   └── sqlmap_useragent/
│
```

---

# Technologies Used

* Snort 3
* Wireshark
* Kali Linux
* Ubuntu Desktop
* VirtualBox
* Nmap
* Hydra
* Hping3
* Dig
* Curl
* FTP Client

---

# Running Snort

```bash
sudo snort \
-c /etc/snort/snort.lua \
-R rules/local.rules \
-i eth0 \
-A alert_fast
```

---

# Test Commands

## SSH Brute Force

```bash
hydra -l test -P passwords.txt ssh://192.168.10.5
```

## SYN Flood

```bash
sudo hping3 -S --flood -p 80 192.168.10.5
```

## High Frequency DNS

```bash
for i in {1..120}; do
    dig test$i.example.com @192.168.10.5
done
```

## Large DNS Query

```bash
dig $(python3 -c "print('A'*60+'.'+'B'*60+'.'+'C'*60)") @192.168.10.5
```

## ICMP Exfiltration

```bash
ping -s 600 192.168.10.5
```

## SYN Scan

```bash
sudo nmap -sS 192.168.10.5
```

## NULL Scan

```bash
sudo nmap -sN 192.168.10.5
```

## XMAS Scan

```bash
sudo nmap -sX 192.168.10.5
```

## FTP Connection

```bash
ftp 192.168.10.5
```

## SQL Injection

```bash
curl "http://192.168.10.5/index.php?id=1%27%20UNION%20SELECT%201"
```

## Command Injection

```bash
curl "http://192.168.10.5/?cmd=ls;whoami"
```

## Directory Traversal

```bash
curl "http://192.168.10.5/../../etc/passwd"
```

## Suspicious User-Agent

```bash
curl -A "sqlmap" http://192.168.10.5/
```

---

# Validation

Each rule was validated using the following workflow:

1. Generate attack traffic from the attacker machine.
2. Capture the traffic using Wireshark.
3. Monitor the network with Snort 3.
4. Verify that the custom rule generates the expected alert.
5. Compare the Snort alert with the captured packets.

Representative screenshots are available in the `screenshots/` directorie.

---

# Results

The implemented rules successfully detected:

* SSH brute-force attempts
* TCP SYN flood attacks
* High-frequency DNS requests
* Large DNS queries (DNS tunneling indicator)
* ICMP-based data exfiltration attempts
* TCP SYN, NULL, and XMAS scans
* Outbound FTP connections
* SQL injection attempts
* Command injection attempts
* Attempts to access `/etc/passwd`
* Suspicious HTTP `User-Agent` values (`sqlmap`)

---

# Future Improvements

* Correlate multiple alerts to improve DNS tunneling detection.
* Reduce false positives using threshold tuning.
* Integrate Snort alerts with a SIEM such as Wazuh or Splunk.
* Add support for malware command-and-control detection.
* Develop additional HTTP and DNS detection rules.

---

# Disclaimer

This project was developed **exclusively for educational and research purposes** within an isolated virtual laboratory. All attack simulations were conducted in a controlled environment without targeting external systems.

---

## ⭐ If you find this project useful, feel free to star the repository!
