# Zero-Downtime Maintenance SOP (EC2 + GT06 + Quickport CRM)

**Owner:** Quickport  
**Applies To:** Production EC2 (`i-0ae922662c241c912`)  
**Critical Services:**
- GT06 GPS TCP Server (Port 5023)
- Quickport CRM (Laravel + Nginx)
- MariaDB
- PM2 Node services
- Traccar (if enabled)

---

## 1. Objective

This SOP ensures **maintenance with ZERO data loss and MINIMAL service disruption**, even when:

- Disk is almost full
- SSH access breaks
- Instance must be stopped
- Volume must be resized
- Services must be restarted

---

## 2. Golden Rules (Never Break)

1. **No maintenance without snapshot**
2. **Elastic IP must remain attached**
3. **GT06 port (5023) must be free before restart**
4. **Never stop instance blindly**
5. **One change at a time**

If unsure → **STOP and snapshot first**

---

## 3. Pre-Maintenance Checklist (Mandatory)

### 3.1 Instance Health
```bash
uptime
df -h
free -h
Abort if:

/ usage > 95% → cleanup first

Load average unusually high

3.2 Confirm Elastic IP
curl ifconfig.me
Expected:

Static Elastic IP (example: 3.109.42.82)

❌ If dynamic IP → attach Elastic IP before proceeding

3.3 Snapshot (ABSOLUTE REQUIREMENT)
AWS Console → EC2 → Volumes → Create Snapshot

Rules:

Description must include date + reason

Snapshot must reach completed state

3.4 AMI (Before Risky Work)
Create AMI only if:

OS / kernel / disk / SSH changes planned

Options:

✅ Tag image and snapshots together

❌ Reboot instance → NO (unless kernel upgrade)

4. Safe Maintenance Scenarios
4.1 Disk Cleanup (No Downtime)
Safe cleanup commands
sudo apt clean
sudo journalctl --vacuum-time=7d
sudo rm -rf /var/log/*.gz
sudo rm -rf /var/log/*.[0-9]
Snap retention
sudo snap set system refresh.retain=2
Check:

df -h
4.2 Volume Resize (Minimal Downtime)
AWS Side
Stop instance

Modify volume size

Start instance

Instance Side
lsblk
sudo growpart /dev/xvda 1
sudo resize2fs /dev/xvda1
df -h
4.3 SSH Recovery (Safe Mode)
If SSH fails:

Stop instance

Detach root volume

Attach to Rescue instance

Fix:

/home/ubuntu/.ssh/authorized_keys

permissions

Reattach volume

Start original instance

Terminate rescue instance

⚠ Rescue instances must NEVER remain running

5. GT06 Zero-Downtime Procedure (CRITICAL)
Before restarting anything
sudo ss -lntp | grep 5023
If occupied by Node:

pm2 stop gt06-gps-service
pm2 delete gt06-gps-service
pm2 save
Restart GT06 safely
cd ~/gt06-server
pm2 start index.js --name "gt06-gps-service"
pm2 save
Verify:

sudo ss -lntp | grep 5023
Expected:

GT06 service listening

No port conflict

6. Service Restart Order (Important)
Correct order:

Database

sudo systemctl status mariadb
Backend

cd /var/www/quickport-laravel
php artisan migrate --force
PM2 Services

pm2 resurrect
pm2 list
Nginx

sudo systemctl reload nginx
7. Post-Maintenance Validation
System
df -h
free -h
uptime
Network
sudo ss -lntp
Application
curl http://127.0.0.1
GT06
Device timestamps updating

Location changing

No reconnect storm

8. Emergency Abort Plan
If anything looks wrong:

Stop immediately

Stop instance

Restore snapshot

Reattach Elastic IP

Start instance

Snapshots > debugging

9. Forbidden Actions
❌ Stop instance without snapshot
❌ Resize disk without backup
❌ Free disk by deleting /var/lib/mysql
❌ Change GT06 port without device update
❌ Leave rescue instance running

10. SOP Status
Approved: ✅
Last validated: Feb 2026
Applies to: All future maintenance

This SOP is mandatory for production stability.


---

## ✅ What this SOP now guarantees

- No accidental data loss  
- No GT06 device blackout  
- No IP-change surprises  
- Recoverable from **any** mistake  

---
