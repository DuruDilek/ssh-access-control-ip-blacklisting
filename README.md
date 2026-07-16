🔐 SSH Access Control and Automated IP Blacklisting


(Fail2Ban + iptables + cron + Email Notifications)


📌 Project Overview

This project was developed to protect Linux systems against SSH brute-force attacks using Fail2Ban, iptables, cron, Bash scripting, and email notifications.

The system automatically detects repeated failed SSH login attempts, temporarily blocks malicious IP addresses, and sends daily email reports to the system administrator.


🎯 Project Objectives

Prevent SSH brute-force attacks.
Automatically ban IP addresses after a specified number of failed login attempts.
Manage banned IP addresses using iptables.
Generate and email daily reports of banned IP addresses to the system administrator.


🛠 Technologies Used

Linux
Fail2Ban
iptables
cron
msmtp (Email Delivery)
Bash Scripting


⚙️ System Architecture

SSH Logs
     ↓
 Fail2Ban
     ↓
iptables (IP Blocking)
     ↓
Bash Script (banned-ips.sh)
     ↓
cron (Daily Scheduled Task)
     ↓
Email Report


🔐 Fail2Ban Configuration (jail.local)
Default Settings
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1
bantime = 180
findtime = 600
maxretry = 3
backend = systemd
usedns = warn
Configuration Details
IP addresses are banned after 3 failed login attempts.
Failed attempts are counted within a 10-minute window.
The ban duration is 3 minutes (180 seconds).
SSH Jail Configuration
[sshd]
enabled = true
port = ssh
logpath = %(sshd_log)s
backend = systemd
findtime = 600
bantime = 180
maxretry = 3
action = %(action_mwl)s

This configuration:

Continuously monitors the SSH service.
Automatically adds banned IP addresses to iptables.
Sends an email notification whenever an IP address is banned.
🔥 IP Blacklisting with iptables

When Fail2Ban detects an SSH brute-force attack, it automatically blocks the offending IP address using:

iptables -A INPUT -s <IP_ADDRESS> -j DROP
📜 Script for Listing Banned IP Addresses

banned-ips.sh

#!/bin/bash
echo "Banned IP Addresses:"
sudo iptables -L -n --line-numbers | grep "DROP"

This script:

Lists all IP addresses blocked by iptables.
Is executed automatically by cron.
⏰ Automated Daily Reporting (cron)
0 8 * * * /home/aysegul/banned-ips.sh | mail -s "Banned IP Addresses" admin@example.com

Every day at 08:00:

The list of banned IP addresses is generated.
A report is emailed to the system administrator.
📧 Email Configuration (msmtp)
defaults
auth on
tls on
tls_trust_file /etc/ssl/certs/ca-certificates.crt

account gmail
host smtp.gmail.com
port 587
from your-email@gmail.com
user your-email@gmail.com
password ********

account default : gmail

Gmail SMTP was used to securely deliver automated email notifications.

🧪 Testing and Validation (PuTTY)

The project was tested using PuTTY to simulate realistic SSH brute-force attack scenarios.

🔹 Test Environment
PuTTY (Windows SSH Client)
🔹 Test Procedure
Opened PuTTY on a Windows machine.
Connected to the Linux server via SSH (Port 22).
Intentionally entered an incorrect password three consecutive times using the same username.
Fail2Ban monitored the SSH logs and detected the repeated failed login attempts.
The attacker's IP address was automatically added to the iptables blacklist.

<img width="793" height="464" alt="resim" src="https://github.com/user-attachments/assets/a01c990c-1ba0-4cfb-9b64-3fba8f1831e4" />


🔹 Results
✔️ The IP address was automatically banned after three failed SSH login attempts.
✔️ The SSH connection from PuTTY was immediately terminated.
✔️ Further access to the server was blocked.
✔️ The IP address was added to the iptables DROP rules.
✔️ The banned IP address was included in the daily email report.


