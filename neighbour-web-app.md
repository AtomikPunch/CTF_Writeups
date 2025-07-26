---
title: "Neighbour - Web Application Enumeration"
description: "A simple web application enumeration challenge involving source code analysis and lateral movement."
date: "2024-01-25"
difficulty: "Easy"
tags:
  - Web Security
  - Source Code Analysis
  - Enumeration
  - Lateral Movement
platform: "TryHackMe"
tools:
  - Browser DevTools
  - Source Code Analysis
  - Web Browser
references:
  - https://tryhackme.com/room/neighbour
  - https://owasp.org/www-project-top-ten/2017/A2_2017-Broken_Authentication
  - https://owasp.org/www-project-top-ten/2017/A5_2017-Broken_Access_Control
slug: neighbour-web-app
---


# 🏠 Neighbour - Web Application Enumeration Write-up

## 🎯 Challenge Overview

**Platform:** TryHackMe  
**Difficulty:** Easy  
**Category:** Web Security & Enumeration

This challenge presents a web application where we need to enumerate user accounts and perform lateral movement to access admin privileges and retrieve the flag.

---

## 🕵️‍♂️ Initial Reconnaissance

### 🌐 Website Access
The challenge starts by accessing the provided URL: \`https://tryhackme.com/room/neighbour\`

Upon visiting the website, we're greeted with a beautiful and well-designed login page. The application appears to be a modern web interface with a clean aesthetic.

![TryHackMe Neighbour Room](/images/ctf/Neighbour/image1.png)

### 🔍 Source Code Analysis
Since we don't have any account credentials initially, the first step is to examine the source code of the page using **Ctrl+U** (View Page Source).

![Login Page](/images/ctf/Neighbour/image2.png)

This reveals a critical vulnerability - the application has hardcoded credentials in the HTML source code!

![Source Code with Credentials](/images/ctf/Neighbour/image4.png)

---

## 🔑 Credential Discovery

### 📝 Hidden Credentials Found
In the page source, we discover the following credentials:

\`\`\`html
<!-- Guest account credentials -->
Login: guest
Password: guest
\`\`\`

This is a classic example of **information disclosure** where sensitive credentials are exposed in the client-side code.

---

## 🚪 Initial Access

### 👤 Guest Account Login
Using the discovered credentials:
- **Username:** \`guest\`
- **Password:** \`guest\`

We successfully log into the application and gain access to the guest profile page.

![Guest Profile Page](/images/ctf/Neighbour/image3.png)

### 🎯 Profile Page Analysis
The guest profile page provides us with:
- User information
- Access to other user profiles
- Potential for lateral movement

---

## 🔄 Lateral Movement

### 👥 User Enumeration
From the guest account, we can now enumerate other users in the system. The application allows us to view different user profiles.

### 👑 Admin Access
Through the lateral movement capabilities, we discover that we can access the **admin user** profile, which contains the flag we're looking for.

![Admin Profile with Flag](/images/ctf/Neighbour/image5.png)

---

## 🏆 Flag Retrieval

### 🎉 Success!
Upon accessing the admin profile, we successfully retrieve the flag and complete the challenge.

---

## 🧠 Key Learning Points

### 🔧 Techniques Used
- **🔍 Source Code Analysis**: Examining HTML source for hidden information
- **👤 User Enumeration**: Discovering user accounts and their relationships
- **🔄 Lateral Movement**: Moving between different user accounts
- **🔑 Credential Discovery**: Finding hardcoded credentials

### 🛡️ Security Vulnerabilities Identified
1. **💬 Information Disclosure**: Credentials exposed in source code
2. **🔐 Weak Authentication**: Hardcoded credentials
3. **👥 Insufficient Access Control**: Ability to access other user profiles
4. **🔍 Poor Input Validation**: No restrictions on user enumeration

---

## 🛡️ Security Implications

### ❌ Anti-Patterns Demonstrated
1. **🔑 Hardcoded Credentials**: Never store credentials in client-side code
2. **👁️ Information Disclosure**: Avoid exposing sensitive data in HTML source
3. **🔓 Weak Access Controls**: Implement proper authorization checks
4. **🔍 No Rate Limiting**: Prevent brute force and enumeration attacks

### ✅ Best Practices
- Use secure authentication mechanisms
- Implement proper session management
- Apply the principle of least privilege
- Regular security audits and code reviews
- Use environment variables for sensitive data

---

## 🎯 Conclusion

This challenge demonstrates the importance of:
- **Thorough enumeration** in web applications
- **Source code analysis** as a reconnaissance technique
- **Lateral movement** concepts in web security
- **Proper access control** implementation

The key lesson is that even simple web applications can have multiple security vulnerabilities that, when chained together, can lead to complete compromise.

**🏆 Challenge Completed Successfully!**

---

*Happy hacking! 🔍✨*
