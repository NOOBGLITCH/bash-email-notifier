# Cron Job + Postfix Gmail SMTP Setup Guide (Ubuntu/Debian)

## Overview

This guide covers:

* Installing and configuring Cron
* Installing and configuring Postfix
* Configuring Gmail SMTP Relay using App Password
* Creating a Bash script that:

  * Creates sequential files (`file1`, `file2`, `file3`, ...)
  * Sends an email notification after file creation
* Scheduling the script using Cron
* Verifying logs and troubleshooting

---

# Step 1: Update System

```bash
sudo apt update
sudo apt upgrade -y
```

---

# Step 2: Install Required Packages

Install Cron:

```bash
sudo apt install cron -y
```

Install Postfix and Mail Utilities:

```bash
sudo apt install postfix mailutils libsasl2-modules -y
```

During Postfix installation:

Select:

```text
Internet Site
```

System Mail Name:

```text
localhost
```

---

# Step 3: Enable Services

Enable and start Cron:

```bash
sudo systemctl enable cron
sudo systemctl start cron
```

Verify:

```bash
systemctl status cron
```

Enable and start Postfix:

```bash
sudo systemctl enable postfix
sudo systemctl restart postfix
```

Verify:

```bash
systemctl status postfix
```

---

# Step 4: Configure Gmail SMTP Relay

Edit Postfix configuration:

```bash
sudo nano /etc/postfix/main.cf
```

Add the following lines:

```ini
relayhost = [smtp.gmail.com]:587

smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous

smtp_tls_security_level = encrypt
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```

Save and exit.

---

# Step 5: Create Gmail Authentication File

Create:

```bash
sudo nano /etc/postfix/sasl_passwd
```

Add:

```text
[smtp.gmail.com]:587 yourgmail@gmail.com:APP_PASSWORD
```

Example:

```text
[smtp.gmail.com]:587 example@gmail.com:abcdabcdabcdabcd
```

---

# Step 6: Secure Credentials

Generate database:

```bash
sudo postmap /etc/postfix/sasl_passwd
```

Secure files:

```bash
sudo chmod 600 /etc/postfix/sasl_passwd
sudo chmod 600 /etc/postfix/sasl_passwd.db
```

---

# Step 7: Restart Postfix

```bash
sudo postfix check
sudo systemctl restart postfix
```

---

# Step 8: Test Email Delivery

```bash
echo "hello from postfix via gmail smtp" | mail -s "test" example@gmail.com
```

Monitor logs:

```bash
sudo tail -f /var/log/mail.log
```

Successful delivery:

```text
status=sent
```

---

# Step 9: Create Working Directory

```bash
mkdir -p /home/linuxusername/Desktop/test
cd /home/linuxusername/Desktop/test
```

---

# Step 10: Create Bash Script

Create script:

```bash
nano /home/linuxusername/Desktop/test/script.sh
```

Paste:

```bash
#!/bin/bash

DIR="/home/linuxusername/Desktop/test"
EMAIL="example@gmail.com"

mkdir -p "$DIR"

cd "$DIR" || exit 1

n=$(find . -maxdepth 1 -name "file*" | wc -l)

FILE="file$((n+1))"

touch "$FILE"

echo "
New file created

File: $FILE
Time: $(date)
Host: $(hostname)
Directory: $DIR
" | mail -s "Cron File Created" "$EMAIL"
```

Save and exit.

---

# Step 11: Make Script Executable

```bash
chmod +x /home/linuxusername/Desktop/test/script.sh
```

Verify:

```bash
ls -l /home/linuxusername/Desktop/test/script.sh
```

Expected:

```text
-rwxr-xr-x
```

---

# Step 12: Test Script Manually

```bash
/home/linuxusername/Desktop/test/script.sh
```

Verify:

```bash
ls -lh /home/linuxusername/Desktop/test
```

Expected:

```text
file1
```

Check Gmail inbox.

---

# Step 13: Create Cron Job

Edit crontab:

```bash
crontab -e
```

Add:

```cron
* * * * * /home/linuxusername/Desktop/test/script.sh >> /home/linuxusername/Desktop/test/cron.log 2>&1
```

Save and exit.

---

# Step 14: Verify Cron Configuration

```bash
crontab -l
```

Expected:

```cron
* * * * * /home/linuxusername/Desktop/test/script.sh >> /home/linuxusername/Desktop/test/cron.log 2>&1
```

---

# Step 15: Monitor Execution

Watch created files:

```bash
watch 'ls -lh /home/linuxusername/Desktop/test'
```

Watch Cron logs:

```bash
tail -f /home/linuxusername/Desktop/test/cron.log
```

Watch Mail logs:

```bash
sudo tail -f /var/log/mail.log
```

Watch Cron service:

```bash
sudo journalctl -u cron -f
```

---

# Useful Commands

Restart Cron:

```bash
sudo systemctl restart cron
```

Restart Postfix:

```bash
sudo systemctl restart postfix
```

Check mail queue:

```bash
mailq
```

Force queue processing:

```bash
sudo postqueue -f
```

View local mailbox:

```bash
mail
```

---

# Expected Result

Every minute:

```text
file1
file2
file3
file4
...
```

An email notification is delivered to:

```text
example@gmail.com
```

Subject:

```text
Cron File Created
```

Workflow:

```text
Cron
  ↓
script.sh
  ↓
Create File
  ↓
Postfix
  ↓
Gmail SMTP
  ↓
Inbox
```
