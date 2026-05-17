# Log Analysis with CLI Tools

## Learning Outcomes
- Use `grep`, `awk`, `sed` to search, extract, and transform log data
- Chain CLI tools for complex log analysis
- Identify suspicious activity in web server logs
- Create professional log analysis reports

## Objective
Master Linux CLI tools (`grep`, `awk`, `sed`, `cut`, `sort`, `uniq`) for security log analysis.

## Scenario
SOC analyst at SecureCorp investigating unusual web traffic. Apache logs provided for analysis to identify attack vectors and compromised systems.

## Prerequisites
- Linux VM (Ubuntu/Kali)
- Basic Apache log format knowledge
- Text editor

## Lab Duration: 2-3 hours

---

## Part 1: Apache Log Format (10 min)

**Combined Log Format:**
```
192.168.1.100 - - [10/Jan/2024:13:55:36 +0000] "GET /index.html HTTP/1.1" 200 2326 "-" "Mozilla/5.0..."
```

| Field | Position | Description |
|-------|----------|-------------|
| IP | $1 | Client IP |
| Identity | $2 | RFC 1413 (usually -) |
| Username | $3 | HTTP auth (usually -) |
| Timestamp | $4 | Request time |
| Request | $5-$7 | Method, URL, Protocol |
| Status | $8 | HTTP response code |
| Size | $9 | Bytes sent |
| Referer | $10 | Referring URL |
| User-Agent | $11+ | Client software |

**Setup:**
```bash
mkdir -p ~/soc-labs/week3 && cd ~/soc-labs/week3
wget https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs -O access.log
head -5 access.log
```

---

## Part 2: Basic Analysis (25 min)

### Exercise 1: Count Requests
```bash
wc -l access.log
```

### Exercise 2: Unique IPs
```bash
# Extract IPs
awk '{print $1}' access.log | sort -u | wc -l

# Alternative
awk '{print $1}' access.log | sort | uniq | wc -l
```

### Exercise 3: Top Talkers
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```
**Analysis:** High counts = heavy users, bots, DDoS, or compromised hosts.

### Exercise 4: Status Codes
```bash
# All codes distribution
awk '{print $9}' access.log | sort | uniq -c | sort -rn

# 404 errors only
awk '$9 == 404' access.log | wc -l

# Better than grep (avoids false matches)
grep " 404 " access.log | wc -l
```

### Exercise 5: Suspicious 404 URLs
```bash
awk '$9 == 404 {print $7}' access.log | sort | uniq -c | sort -rn | head -5
```
**Findings:** `/admin/`, `/.env`, `/phpMyAdmin/` = recon for admin panels, sensitive files.

---

## Part 3: Pattern Matching with grep (35 min)

### Exercise 6: Detect Scanners
```bash
# Nikto scanner
grep -i "nikto" access.log | wc -l
grep -i "nikto" access.log | awk '{print $1}' | sort -u

# Multiple scanners
grep -iE "(nikto|nessus|nmap|sqlmap|burp|wpscan|acunetix)" access.log
```

### Exercise 7: SQL Injection
```bash
# Keywords
grep -iE "(union|select|insert|update|delete|drop|exec)" access.log | wc -l

# URL-encoded
grep -E "(%27|%20union|%20select)" access.log

# Extract attacker IPs and URLs
grep -iE "(union|select)" access.log | awk '{print $1, $7}' | head -10

# Count by IP
grep -iE "(union|select)" access.log | awk '{print $1}' | sort | uniq -c | sort -rn
```

### Exercise 8: XSS Detection
```bash
grep -iE "(<script|javascript:|onerror=|%3Cscript)" access.log | wc -l

# Identify attackers
grep -iE "(<script|javascript:)" access.log | awk '{print $1}' | sort | uniq -c | sort -rn
```

### Exercise 9: Directory Traversal
```bash
grep -E "(\.\./|\.\.%2[Ff])" access.log | wc -l

# Extract targets
grep -E "(\.\./|\.\.%2[Ff])" access.log | awk '{print "IP: "$1", Target: "$7}'
```

### Exercise 10: Command Injection
```bash
grep -iE "(;|\||%3B|%7C)\s*(ls|cat|id|whoami|wget|curl|bash|nc)" access.log

# Count by attacker
grep -iE ";.*(ls|cat|id|whoami)" access.log | awk '{print $1}' | sort | uniq -c | sort -rn
```

---

## Part 4: Advanced awk (35 min)

### Exercise 11: Traffic by Hour
```bash
awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c | sort -rn
```
**Analysis:** Spikes at odd hours (3 AM) suggest automated attacks.

### Exercise 12: HTTP Methods
```bash
awk '{print $6}' access.log | tr -d '"' | sort | uniq -c | sort -rn
```
**Note:** Unusual methods (PUT, DELETE, TRACE) indicate attacks.

### Exercise 13: POST Requests
```bash
# Unique IPs making POST
awk '$6 == "\"POST" {print $1}' access.log | sort -u

# POST details
awk '$6 == "\"POST" {print $1, $7, $9}' access.log | head -10

# Suspicious POSTs
awk '$6 == "\"POST"' access.log | grep -iE "(login|admin|upload|cmd|exec)"
```

### Exercise 14: Bandwidth
```bash
# Total bandwidth
awk '{sum += $10} END {print sum/1024/1024 " MB"}' access.log

# Top consumers
awk '{bytes[$1] += $10} END {for (ip in bytes) printf "%s %.2f MB\n", ip, bytes[ip]/1024/1024}' access.log | sort -k2 -rn | head -10
```

### Exercise 15: Response Size Analysis
```bash
# Smallest (possible errors/probes)
awk '{print $1, $7, $10}' access.log | sort -k3 -n | head -10

# Largest (possible exfiltration)
awk '{print $1, $7, $10}' access.log | sort -k3 -rn | head -10

# Average per IP
awk '{sum[$1]+=$10; count[$1]++} END {for (ip in sum) print ip, sum[ip]/count[ip]}' access.log | sort -k2 -rn
```

### Exercise 16: User Agents
```bash
# Most common
awk -F'"' '{print $6}' access.log | sort | uniq -c | sort -rn | head -10

# Suspicious agents
awk -F'"' '{print $6}' access.log | grep -iE "(nikto|sqlmap|curl|wget|python|scanner|bot)" | sort -u
```

---

## Part 5: Text Transformation with sed (25 min)

### Exercise 17: Extract Unique IPs
```bash
awk '{print $1}' access.log | sort -u > unique_ips.txt
wc -l unique_ips.txt
```

### Exercise 18: Anonymize IPs
```bash
# Replace last octet with XXX
sed 's/\.[0-9]*\( \)/\.XXX\1/' access.log | head -5

# awk alternative
awk '{sub(/[0-9]+$/, "XXX", $1); print}' access.log | head -5
```

### Exercise 19: Extract URLs
```bash
# All unique URLs
awk '{print $7}' access.log | sort -u > urls.txt

# Sensitive file types only
awk '{print $7}' access.log | grep -E "\.(php|asp|jsp|cgi|env|bak|sql)$" | sort -u

# URLs with parameters (injection points)
awk '{print $7}' access.log | grep '\?' | sort -u > urls-with-params.txt
```

### Exercise 20: Format Timestamps
```bash
# Remove brackets
awk '{print $4}' access.log | sed 's/\[//;s/\]//' | head -10

# Date only
awk '{print $4}' access.log | sed 's/\[//;s/:.*//' | head -10
```

### Exercise 21: Clean Output
```bash
# IP, timestamp, URL only
awk '{print $1, $4, $7}' access.log | sed 's/\[//;s/\]//' | head -10
```

---

## Part 6: Complex Pipelines (25 min)

### Exercise 22: Per-IP Activity Report
```bash
for ip in $(awk '{print $1}' access.log | sort -u); do
    echo "=== $ip ==="
    echo "Requests: $(grep -c "^$ip " access.log)"
    echo "404s: $(grep "^$ip " access.log | awk '$9==404' | wc -l)"
    echo "SQLi: $(grep -iE "(union|select)" access.log | grep -c "^$ip ")"
    echo "XSS: $(grep -iE "(<script|javascript:)" access.log | grep -c "^$ip ")"
    echo ""
done > suspicious-activity.txt
```

### Exercise 23: Threat Scoring
```bash
awk '{
    ip = $1; score = 0;
    if ($9 == 500) score += 2;
    if ($9 == 403) score += 1;
    if (tolower($0) ~ /union|select|drop/) score += 1;
    if (tolower($0) ~ /<script|javascript:/) score += 1;
    if ($0 ~ /\.\.\//) score += 1;
    scores[ip] += score;
} END {
    for (ip in scores) 
        if (scores[ip] > 0) print ip, scores[ip]
}' access.log | sort -k2 -rn > threat-scores.txt
```

### Exercise 24: Scan Detection
```bash
# IPs with most unique 404 URLs (scanning behavior)
awk '$9 == 404 {print $1, $7}' access.log | sort -u | awk '{print $1}' | uniq -c | sort -rn | head -10

# High 404 ratio
for ip in $(awk '{print $1}' access.log | sort -u); do
    total=$(grep -c "^$ip " access.log)
    notfound=$(grep "^$ip " access.log | awk '$9==404' | wc -l)
    if [ $total -gt 20 ]; then
        echo "$ip - Total: $total, 404s: $notfound, Ratio: $((notfound*100/total))%"
    fi
done | sort -t: -k3 -rn
```

---

## Part 7: Analysis Report Template (20 min)

Create `Log-Analysis-Report.md`:

```markdown
# Apache Log Analysis Report
**Analyst:** [Name]
**Date:** [Date]
**Log:** access.log

## Executive Summary
[2-3 sentence key findings summary]

## 1. Traffic Overview
- **Total Requests:** [wc -l result]
- **Unique IPs:** [sort -u | wc -l result]
- **Top 10 IPs:** [paste top talkers output]

## 2. Status Codes
- **Distribution:** [paste status code output]
- **Total 404s:** [count]
- **Top 5 404 URLs:** [paste output]
- **Analysis:** [interpretation]

## 3. Security Findings
### Scanner Activity (Nikto)
- Count: [number]
- Source IPs: [list]

### SQL Injection
- Attempts: [count]
- Attackers: [paste by-IP output]
- Samples: [2-3 example requests]

### XSS
- Attempts: [count]

### Directory Traversal
- Attempts: [count]

### Command Injection
- Attempts: [count]

## 4. Methods Analysis
[paste HTTP methods output]

## 5. High-Risk IPs
| IP | Requests | Activities | Threat Score |
|----|----------|------------|--------------|
| [IP] | [count] | [SQLi, XSS, scan] | [score] |

## 6. Recommendations
- **Block:** [IPs] - [reasons]
- **Investigate:** [specific findings]
- **Patch:** [vulnerabilities]

## 7. Bandwidth
- **Total:** [X MB]
- **Top Consumers:** [paste output]

## Conclusion
[Summary and overall security posture]

## Appendices
### Commands Used
[list all commands]
### Suspicious IPs
[paste suspicious_ips.txt]
```

---

## Deliverables
1. `Log-Analysis-Report.md` - Complete report
2. `unique_ips.txt` - All unique IPs
3. `suspicious_ips.txt` - Attack IPs
4. `urls.txt` - All requested URLs
5. `threat-scores.txt` - Threat scores
6. `sql-injection-attempts.log` - SQLi attempts

---

## Evaluation
| Criterion | Weight |
|-----------|--------|
| Completeness | 25% |
| Command Accuracy | 25% |
| Analysis Quality | 25% |
| Documentation | 15% |
| Security Awareness | 10% |

---

## Optional Challenges
1. Automate entire analysis in a bash script
2. Compare multiple log files for trends
3. Write SIEM detection rules from findings
4. One-liner to find most dangerous IP
5. Generate attack timeline sorted chronologically
6. Correlate IPs with AbuseIPDB/VirusTotal

---

## Quick Reference

### grep
```bash
grep -i "pattern" file          # case-insensitive
grep -E "regex" file             # extended regex
grep -v "pattern" file           # invert match
grep -c "pattern" file           # count matches
grep -A 2 "pattern" file         # 2 lines after match
grep -B 2 "pattern" file         # 2 lines before
```

### awk
```bash
awk '{print $1}' file            # print field 1
awk '$3 > 100' file              # filter by condition
awk -F':' '{print $1}' file      # custom delimiter
awk '{sum+=$1} END {print sum}'  # sum fields
```

### sed
```bash
sed 's/old/new/' file            # replace first occurrence
sed 's/old/new/g' file           # replace all
sed '/pattern/d' file            # delete matching lines
sed -n '5,10p' file              # print lines 5-10
```

### sort & uniq
```bash
sort file                        # alphabetical sort
sort -n file                     # numerical sort
sort -r file                     # reverse sort
sort -u file                     # unique sort
uniq -c file                     # count occurrences
