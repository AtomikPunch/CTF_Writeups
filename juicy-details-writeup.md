---
title: "Juicy Details"
description: "Analyzing logs to trace attacker activities in the Juicy Details room on TryHackMe."
date: "2025-07-27"
difficulty: "Easy"
tags:
  - Log Analysis
  - Forensics
  - CTF
  - Web Exploitation
platform: "TryHackMe"
tools:
  - Nmap
  - Hydra
  - Sqlmap
  - curl
  - Feroxbuster
references:
  - https://tryhackme.com/room/juicydetails
  - https://nmap.org/book/nse.html
  - http://sqlmap.org
slug: juicy-details-log-analysis
---


# TryHackMe - Juicy Details Room: Log Analysis Write-up

Link to the room: [Juicy Details on TryHackMe](https://tryhackme.com/room/juicydetails)

This room offers a compelling opportunity to enhance investigative skills, particularly around log analysis and reconnaissance tools.

## Tools Identified in the Attack

The attacker used a sequence of five tools, some of which were familiar, while others were discovered during this room.

---

### 1. **Nmap**
```
::ffff:192.168.10.5 - - [11/Apr/2021:09:08:35 +0000] "POST / HTTP/1.1" 200 1924 "-" "Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)"
```

Nmap (Network Mapper) is an open-source tool used for network discovery and security auditing. It can identify devices on a network, detect open ports, and determine operating systems and services in use.

---

### 2. **Hydra**
```
::ffff:192.168.10.5 - - [11/Apr/2021:09:16:27 +0000] "GET /rest/user/login HTTP/1.0" 500 - "-" "Mozilla/5.0 (Hydra)"
```

Hydra is a powerful, open-source login cracker designed to perform brute-force and dictionary attacks efficiently.

---

### 3. **Sqlmap**
```
::ffff:192.168.10.5 - - [11/Apr/2021:09:29:14 +0000] "GET /rest/products/search?q=1 HTTP/1.1" 200 - "-" "sqlmap/1.5.2#stable (http://sqlmap.org)"
```

Sqlmap is a highly automated penetration testing tool designed to detect and exploit SQL injection vulnerabilities in web applications.

---

### 4. **Curl**
```
::ffff:192.168.10.5 - - [11/Apr/2021:09:32:51 +0000] "GET /rest/products/search?q=qwert%27))%20UNION%20SELECT%20id,%20email,%20password,%20%274%27,%20%275%27,%20%276%27,%20%277%27,%20%278%27,%20%279%27%20FROM%20Users-- HTTP/1.1" 200 3742 "-" "curl/7.74.0"
```

Curl is a versatile command-line tool used for transferring data using URLs. It is commonly used for testing APIs, downloading files, and debugging HTTP requests.

---

### 5. **Feroxbuster**
```
::ffff:192.168.10.5 - - [11/Apr/2021:09:34:33 +0000] "GET /login HTTP/1.1" 200 1924 "-" "feroxbuster/2.2.1"
```

Feroxbuster is a fast, recursive content discovery tool written in Rust, designed to uncover hidden directories and files within web applications via wordlist-based forced browsing.

---

## Questions & Answers

### **What tools did the attacker use?**  
**Answer:** Nmap, Hydra, Sqlmap, Curl, Feroxbuster

---

### **What endpoint was vulnerable to a brute-force attack?**  
**Answer:** `/rest/user/login`

This endpoint appeared multiple times in the logs, indicating attempts to brute-force login credentials.

---

### **What endpoint was vulnerable to SQL injection?**  
**Answer:** `/rest/products/search`

SQL injection attempts are evident in lines such as:
```
::ffff:192.168.10.5 - - [11/Apr/2021:09:29:16 +0000] "GET /rest/products/search?q=1%20AND%20EXTRACTVALUE(7542,CONCAT(0x5c,0x717a6b7171,(SELECT (ELT(7542=7542,1))),0x716a787071))-- GMaz HTTP/1.1" 200 30 "-" "sqlmap/1.5.2#stable (http://sqlmap.org)"
```

This confirms that the parameter `q` is injectable.

---

### **What endpoint did the attacker try to use to retrieve files?**  
**Answer:** `/ftp`

Example log line:
```
::ffff:192.168.10.5 - - [11/Apr/2021:09:34:33 +0000] "GET /ftp HTTP/1.1" 200 4852 "-" "feroxbuster/2.2.1"
```

---

### **What section of the website did the attacker use to scrape user email addresses?**  
**Answer:** Product review section  
```
::ffff:192.168.10.5 - - [11/Apr/2021:09:09:23 +0000] "GET /rest/products/1/reviews HTTP/1.1" 200 172 "http://192.168.10.4/" "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0"
```

Review sections often include user emails, which can be scraped by attackers.

---

### **What user information was the attacker able to retrieve from the vulnerable SQL injection endpoint?**  
**Answer:** User IDs, emails, and passwords  
```
::ffff:192.168.10.5 - - [11/Apr/2021:09:32:51 +0000] "GET /rest/products/search?q=qwert')) UNION SELECT id, email, password, '4', '5', '6', '7', '8', '9' FROM Users-- HTTP/1.1" 200 3742 "-" "curl/7.74.0"
```

---

### **What files did they try to download from the vulnerable endpoint?**  
**Answer:** `/www-data.bak`, `/coupons_2013.md.bak`  
From the `vsftpd.log`:
```
Sun Apr 11 09:35:45 2021 [pid 8154] [ftp] OK DOWNLOAD: Client "::ffff:192.168.10.5", "/www-data.bak", 2602 bytes, 544.81Kbyte/sec
Sun Apr 11 09:36:08 2021 [pid 8154] [ftp] OK DOWNLOAD: Client "::ffff:192.168.10.5", "/coupons_2013.md.bak", 131 bytes, 3.01Kbyte/sec
```

---

### **What service and account name were used to retrieve the files?**  
**Answer:** `ftp`, username: `anonymous`  
```
Sun Apr 11 08:18:07 2021 [pid 6627] [ftp] OK LOGIN: Client "::ffff:127.0.0.1", anon password "ls"
```

---

### **What service and username were used to gain shell access to the server?**  
**Answer:** `ssh`, username: `www-data`  
```
Apr 11 09:39:37 thunt sshd[8232]: Failed password for www-data from 192.168.10.5 port 40084 ssh2
```

---

## Final Thoughts

This TryHackMe room provided a concise and practical overview of log analysis techniques. It was an excellent opportunity to sharpen investigative skills and discover or revisit several powerful tools used in cybersecurity assessments.

