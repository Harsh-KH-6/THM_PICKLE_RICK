# THM_PICKLE_RICK


# Phase 1 – Enumeration

1. Nmap Scan

We started with basic service enumeration:

nmap -sC -sV -Pn -oN initial_scan.txt <TARGET_IP>

Results:

Port 22 → SSH (OpenSSH 8.2p1)

Port 80 → Apache Web Server


<img width="1560" height="506" alt="image" src="https://github.com/user-attachments/assets/820b7c6a-7ffb-43e5-9558-a07fbf96134c" />

Since a web service was available, we focused on HTTP.


2. Web Enumeration

Visiting the homepage revealed a themed page.

Viewing Page Source

Inside HTML comments we discovered:

Username: R1ckRul3s

<img width="1412" height="643" alt="image" src="https://github.com/user-attachments/assets/b43d793c-3102-4a72-9995-06cb5aab6701" />

This shows why viewing source code is critical during testing.


3. robots.txt Discovery

Checking:

/robots.txt

Revealed:

Wubbalubbadubdub

<img width="1178" height="183" alt="image" src="https://github.com/user-attachments/assets/111cf256-5bf8-4550-9513-0dd19a42f3e7" />


This was used as the password for the login panel.

---

# Phase 2 – Exploitation

4. Login Panel Access

Using:

Username: R1ckRul3s

Password: Wubbalubbadubdub

We gained access to a Command Panel.

<img width="1420" height="647" alt="image" src="https://github.com/user-attachments/assets/a980d223-d9a8-4425-9e6f-83df9f5ddb1a" />

<img width="1413" height="497" alt="image" src="https://github.com/user-attachments/assets/82a80cba-f86f-432f-a051-d6f3bf6080af" />

5. Command Injection

Testing basic commands:

1) ls

2) whoami

3) pwd

Confirmed:

User: www-data

Working Directory: /var/www/html

This confirmed Remote Command Execution (RCE).

6. Bypassing Command Filters

The cat command was blocked.

<img width="990" height="304" alt="image" src="https://github.com/user-attachments/assets/79b167f4-36d2-48bb-ac5a-b18c25314f17" />


Instead of stopping, we bypassed the blacklist using:

less filename

This allowed us to retrieve:

First ingredient (web directory)

This demonstrates why blacklist-based filtering is insecure.

---

# Phase 3 – Internal Enumeration

The clue file said:

Look around the file system

We explored:

ls /
ls /home

Discovered:

/home/rick

Inside:

second ingredients

<img width="1314" height="132" alt="image" src="https://github.com/user-attachments/assets/e72b4487-3142-4d08-9882-be8dd3abca5d" />


Using:

less "/home/rick/second ingredients"

We retrieved the second ingredient.

---

# Phase 4 – Privilege Escalation

Running:

sudo -l

<img width="1324" height="191" alt="image" src="https://github.com/user-attachments/assets/584a5f1d-5705-427b-b7c2-b7ef4679575a" />

Revealed:

(ALL) NOPASSWD: ALL

This is a critical sudo misconfiguration.

It means:

The www-data user can execute ANY command as root without a password.

We escalated privileges using:

sudo ls /root
sudo less /root/3rd.txt

Successfully retrieving the final ingredient.

---

# Vulnerabilities Identified

-Information disclosure in HTML comments

-Sensitive data in robots.txt

-Command Injection vulnerability

-Weak blacklist-based filtering

-Over-permissive file permissions (777)

-Misconfigured sudo privileges (Privilege Escalation)

---

# Key Learnings

-Always enumerate before exploiting

-Source code inspection is powerful

-robots.txt often leaks information

-Blacklisting commands is ineffective

-Internal enumeration is crucial

-Misconfigured sudo is extremely dangerous
