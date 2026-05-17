# Web Server Log Investigation Lab

## SOC Training - Apache Access Log Analysis

**Level:** Beginner  
**Tools:** grep, awk, sort, uniq, wc  
**Objective:** Learn to investigate Apache access logs and identify suspicious activity.

---

# Log Format

Apache Combined Log Format:
```
%h %l %u %t "%r" %>s %b "%{Referer}i" "%{User-Agent}i"
```

Sample entry:
```
203.0.113.25 - - [12/Feb/2026:09:15:42 +0000] "GET /dashboard HTTP/1.1" 200 4521 "-" "Mozilla/5.0"
```

| Field | Position | Description |
|-------|----------|-------------|
| IP | $1 | Client IP address |
| Timestamp | $4 | Date & time |
| Method | $6 | HTTP method (GET/POST) |
| URL | $7 | Requested resource |
| Status | $9 | HTTP response code |
| User-Agent | $12+ | Browser/tool identifier |

---

# Lab Setup

```bash
mkdir -p ~/security-labs/apache-analysis
cd ~/security-labs/apache-analysis

# Download sample log
wget https://raw.githubusercontent.com/elastic/examples/master/Common%20Data%20Formats/apache_logs/apache_logs -O access.log

# Verify
head -5 access.log
wc -l access.log
```

---

# Basic Analysis

## 1. Count Total Requests
```bash
wc -l access.log
```

## 2. Extract Unique IPs
```bash
awk '{print $1}' access.log | sort -u | wc -l
```

## 3. Top Active IPs
```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10
```

## 4. HTTP Status Code Distribution
```bash
awk '{print $9}' access.log | sort | uniq -c | sort -nr
```

## 5. Top Missing Pages (404)
```bash
awk '$9 == 404 {print $7}' access.log | sort | uniq -c | sort -nr | head -10
```

## 6. HTTP Method Distribution
```bash
awk '{print $6}' access.log | tr -d '"' | sort | uniq -c | sort -nr
```

---

# Threat Hunting

## 7. Detect Vulnerability Scanners
```bash
# Find scanner user-agents
grep -i "nikto\|nmap\|sqlmap\|dirb\|gobuster" access.log

# Extract scanner IPs
grep -i "nikto\|nmap\|sqlmap" access.log | awk '{print $1}' | sort -u
```

## 8. SQL Injection Attempts
```bash
# Search for SQLi patterns
grep -iE "(union.*select|select.*from|drop table|--|';)" access.log

# Count attempts
grep -ciE "(union|select|drop|insert)" access.log

# Get attacking IPs
grep -iE "(union|select|drop)" access.log | awk '{print $1}' | sort | uniq -c | sort -nr
```

## 9. Cross-Site Scripting (XSS)
```bash
# Search for XSS patterns
grep -iE "(<script|javascript:|alert\(|onerror=|onload=)" access.log

# Count XSS attempts
grep -ciE "(<script|alert\(|javascript:)" access.log
```

## 10. Directory Traversal
```bash
# Search for traversal patterns
grep -E "(\.\./|\.\.%2[fF]|\.\.%252[fF])" access.log

# Count attempts
grep -cE "(\.\./|\.\.%2[fF])" access.log
```

## 11. Suspicious User-Agents
```bash
# Top user-agents
awk -F'"' '{print $6}' access.log | sort | uniq -c | sort -nr | head -10

# Empty or suspicious UAs
grep '"-"' access.log | awk '{print $1}' | sort -u
```

---

# Advanced Analysis

## 12. Traffic by Hour
```bash
# Identify traffic spikes
awk '{print $4}' access.log | cut -d: -f2 | sort | uniq -c | sort -nr
```

## 13. POST Request Analysis
```bash
# Count POST requests
grep -c '"POST' access.log

# POST requests by IP
awk '$6 == "\"POST"' access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -10
```

## 14. Bandwidth Usage
```bash
# Total data transferred
awk '{sum += $10} END {print sum/1024/1024 " MB"}' access.log

# Top bandwidth consumers
awk '{traffic[$1] += $10} END {for (ip in traffic) print ip, traffic[ip]/1024/1024 " MB"}' access.log | sort -k2 -nr | head -10
```

## 15. Suspicious URL Patterns
```bash
# URLs with unusual length (buffer overflow attempts)
awk '{print length($7), $1, $7}' access.log | sort -nr | head -10

# Dynamic page requests
awk '{print $7}' access.log | grep -E "\.(php|jsp|asp|cgi)" | sort -u
```

---

# Reporting Commands

## Extract Data for Report
```bash
# Unique IPs
awk '{print $1}' access.log | sort -u > unique_ips.txt

# All URLs
awk '{print $7}' access.log | sort -u > urls.txt

# Suspicious IPs (combine multiple indicators)
grep -iE "(union|select|<script|nikto|\.\./)" access.log | awk '{print $1}' | sort -u > suspicious_ips.txt

# 404 errors with URLs
awk '$9 == 404 {print $1, $7}' access.log > not_found_requests.txt
```

---

# One-Liner Threat Hunt

```bash
# Comprehensive check for all attack indicators
grep -iE "(union|select|<script|\.\./|nikto|nmap|sqlmap|drop|alert)" access.log | \
    awk '{print $1, $7, $9}' | \
    sort -u | \
    tee threat_hits.txt
```

---

# Investigation Report Template

```bash
cat > report.md << 'EOF'
# Apache Log Investigation Report

## Summary
- **Total Requests:** `wc -l < access.log`
- **Unique IPs:** `awk '{print $1}' access.log | sort -u | wc -l`
- **Total Bandwidth:** `awk '{sum+=$10} END {print sum/1024/1024 " MB"}' access.log`

## Top 10 IPs by Activity
```
awk '{print $1}' access.log | sort | uniq -c | sort -nr | head -10
```

## Status Code Summary
```
awk '{print $9}' access.log | sort | uniq -c | sort -nr
```

## Security Findings

### SQL Injection
- Attempts: `grep -ciE "(union|select|drop)" access.log`
- Attacking IPs: `grep -iE "(union|select)" access.log | awk '{print $1}' | sort -u`

### XSS Attempts
- Attempts: `grep -ciE "(<script|alert\(|javascript:)" access.log`

### Directory Traversal
- Attempts: `grep -cE "(\.\./|\.\.%2[fF])" access.log`

### Scanner Activity
- Detected scanners: `grep -ciE "(nikto|nmap|sqlmap)" access.log`
- Scanner IPs: `grep -iE "(nikto|nmap)" access.log | awk '{print $1}' | sort -u`

## Suspicious IPs to Block
```
grep -iE "(union|select|<script|nikto|\.\./)" access.log | awk '{print $1}' | sort -u
```

## Recommendations
1. Block identified malicious IPs
2. Implement WAF rules for SQLi/XSS
3. Monitor repeated 404 errors
4. Review high-volume IPs for rate limiting
EOF
```

---

# Quick Reference Card

| Task | Command |
|------|---------|
| Count requests | `wc -l access.log` |
| Unique IPs | `awk '{print $1}' access.log \| sort -u \| wc -l` |
| Top 10 IPs | `awk '{print $1}' access.log \| sort \| uniq -c \| sort -nr \| head` |
| 404 errors | `awk '$9 == 404' access.log \| wc -l` |
| SQLi attempts | `grep -ciE "(union\|select)" access.log` |
| XSS attempts | `grep -ciE "(<script\|alert)" access.log` |
| Scanner IPs | `grep -i "nikto" access.log \| awk '{print $1}' \| sort -u` |
| Hourly traffic | `awk '{print $4}' access.log \| cut -d: -f2 \| sort \| uniq -c` |
| POST requests | `grep -c '"POST' access.log` |
| Bandwidth total | `awk '{sum+=$10} END {print sum/1024/1024 " MB"}' access.log` |

---

# Deliverables

Submit these files:
- `unique_ips.txt`
- `urls.txt`
- `suspicious_ips.txt`
- `report.md`

---

# Bonus Challenges

1. **Block IPs script:**
```bash
grep -iE "(union|select|<script)" access.log | awk '{print $1}' | sort -u | while read ip; do
    sudo ufw deny from $ip
done
```

2. **Real-time monitoring:**
```bash
tail -f access.log | while read line; do
    echo "$line" | grep -qiE "(union|select|<script|\.\./)" && echo "ALERT: $line"
done
```

3. **Hourly spike detection:**
```bash
for hour in {00..23}; do
    count=$(grep -c ":${hour}:" access.log)
    [ $count -gt 1000 ] && echo "Hour $hour: $count requests (SPIKE)"
done
```

---

# Estimated Completion Time: 1-2 Hours

After this lab, you can analyze web logs, detect attacks, and create investigation reports.
```
