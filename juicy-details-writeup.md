# 🕵️ Juicy Details – TryHackMe CTF Write-Up

**Room:** [Juicy Details](https://tryhackme.com/room/juicydetails)  
**Focus:** Log analysis, tool identification, endpoint enumeration

---

## 🛠️ Tools Used by the Attacker (In Order of Appearance)

1. **Nmap**  
   Used for network scanning and service detection.  
   **User-Agent:**
   ```
   Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)
   ```

2. **Hydra**  
   Utilized for brute-force attacks on login forms.  
   **User-Agent:**
   ```
   Mozilla/5.0 (Hydra)
   ```

3. **Sqlmap**  
   Automated SQL injection testing tool.  
   **User-Agent:**
   ```
   sqlmap/1.5.2#stable (http://sqlmap.org)
   ```

4. **curl**  
   Used to manually send a malicious SQL injection payload.  
   **User-Agent:**
   ```
   curl/7.74.0
   ```

5. **Feroxbuster**  
   Recursive content discovery tool.  
   **User-Agent:**
   ```
   feroxbuster/2.2.1
   ```

---

## 🔐 Brute-Force Attack Target

**Vulnerable Endpoint:**  
```
/rest/user/login
```

Detected from:
```
"GET /rest/user/login HTTP/1.0" 500 - "-" "Mozilla/5.0 (Hydra)"
```

---

## 💉 SQL Injection Vulnerability

**Vulnerable Endpoint:**  
```
/rest/products/search?q=1 AND EXTRACTVALUE(...)
```

**Parameter:** `q`

SQL injection tools used: sqlmap and curl.

---

## 📁 File Retrieval Attempt

**Endpoint Accessed:**  
```
/ftp
```

Feroxbuster attempted to enumerate this path to download files.

---

## 📧 Scraping User Emails

**Endpoint Accessed:**  
```
/rest/products/1/reviews
```

User emails were visible in product reviews.

---

## 🧾 Data Retrieved via SQL Injection

**Payload Example:**
```sql
UNION SELECT id, email, password, '4', '5', '6', '7', '8', '9' FROM Users--
```

**Data Extracted:**
- User IDs
- Emails
- Passwords

---

## 📦 Files Downloaded

From the FTP logs:
- `/www-data.bak`
- `/coupons_2013.md.bak`

**Log Evidence:**
```
OK DOWNLOAD: Client "::ffff:192.168.10.5", "/www-data.bak", 2602 bytes
OK DOWNLOAD: Client "::ffff:192.168.10.5", "/coupons_2013.md.bak", 131 bytes
```

---

## 🔓 FTP Access Details

- **Service:** FTP  
- **Username:** `anonymous`  

**Log Evidence:**
```
OK LOGIN: Client "::ffff:127.0.0.1", anon password "ls"
```

---

## 🖥️ Gaining Shell Access

- **Service:** SSH  
- **Username:** `www-data`  

**Log Evidence:**
```
Failed password for www-data from 192.168.10.5 port 40084 ssh2
```

---

## 📝 Summary

This TryHackMe room focused on log analysis and gave insight into:
- Identifying attacker tools and techniques
- Tracking endpoint vulnerabilities
- Understanding data exfiltration
- Observing real-world attack patterns in logs

Perfect for sharpening Blue Team skills!

---