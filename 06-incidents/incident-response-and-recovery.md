# Incident Response & Recovery Playbook

**System:** Quickport CRM + GT06 Tracking  
**Environment:** Production  
**AWS Region:** ap-south-1  
**Severity Coverage:** Low → Critical  
**Last Verified:** Feb 2026  

---

## 1. Purpose

This document defines **exact actions** to take when something breaks.

Goals:
- No panic
- No guesswork
- No irreversible damage
- Fast, confident recovery

If you follow this playbook, **every incident is survivable**.

---

## 2. Incident Severity Levels

| Level | Description | Examples |
|---|---|---|
| 🟢 Low | Minor service glitch | UI issue, delayed GPS |
| 🟡 Medium | Partial outage | API down, GT06 not connecting |
| 🔴 High | Core service down | DB failure, disk full |
| ⚫ Critical | Data risk | Corruption, accidental delete |

---

## 3. Universal First Rule (ALL Incidents)

🚨 **STOP BEFORE FIXING**

Always do this first:

1. **Freeze changes**
2. **Identify scope**
3. **Create snapshot**

```text
NO SNAPSHOT = NO ACTION
4. Immediate Safety Checklist
Run before touching anything:

date
uptime
df -h
free -h
AWS:

Instance state: running

Volume status: attached

Elastic IP: associated

5. Incident Playbooks
5.1 GT06 Devices Not Connecting
Symptoms
Devices offline

No new location updates

TCP port silent

Diagnosis
sudo ss -lntp | grep 5023
pm2 list
Fix Steps
Check port owner:

sudo ss -lntp | grep 5023
Kill conflicting process:

sudo pkill node
Start GT06 service:

cd ~/gt06-server
pm2 start index.js --name gt06-gps-service
pm2 save
Verify:

pm2 list
sudo ss -lntp | grep 5023
✅ Expected:

Port 5023 owned by node

PM2 shows gt06-gps-service

5.2 Disk Full / System Slowness
Symptoms
SSH delays

Services fail to start

DB errors

Diagnosis
df -h
sudo du -xh /var --max-depth=2 | sort -h | tail -20
Immediate Relief
sudo apt clean
sudo journalctl --vacuum-time=7d
sudo rm -rf /var/log/*.gz
Permanent Fix
Increase EBS volume

Extend filesystem (online)

sudo growpart /dev/xvda 1
sudo resize2fs /dev/xvda1
5.3 Database Not Responding
Symptoms
500 errors

Laravel DB connection errors

Diagnosis
sudo systemctl status mysql
Fix
sudo systemctl restart mysql
If corruption suspected:

STOP

Snapshot

Restore DB from dump

5.4 Website/API Down
Diagnosis
sudo systemctl status nginx
curl -I http://127.0.0.1
Fix
sudo systemctl restart nginx
php artisan optimize:clear
5.5 SSH Login Fails
Emergency Recovery
Stop instance

Attach root volume to rescue EC2

Mount volume

Fix:

/home/ubuntu/.ssh/authorized_keys

Permissions:

chmod 700 ~/.ssh
chmod 600 authorized_keys
Reattach volume

Start instance

6. Data Corruption / Accidental Deletion (CRITICAL)
🚨 DO NOT RESTART SERVICES

Steps:

Stop instance

Snapshot volume

Assess scope

Restore from:

Snapshot (preferred)

DB dump (if partial)

Never attempt “quick fixes” here.

7. Recovery Validation Checklist
After any recovery:

 GT06 devices reporting

 CRM loads

 DB queries OK

 Disk usage < 80%

 PM2 services stable

 Elastic IP attached

8. Incident Log (Mandatory)
Record:

Time

Root cause

Action taken

Preventive step

Store in:

/docs/incidents/YYYY-MM-DD.md
9. Rules That Must Never Be Broken
❌ Fix without snapshot
❌ Restart blindly
❌ Delete data to “free space”
❌ Change infra during incident

10. Mental Model
Incidents are controlled events, not emergencies.
Calm execution beats speed.

End of Document


---
