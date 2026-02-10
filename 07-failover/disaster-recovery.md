# Full EC2 Recovery Playbook (Production)

**System:** Quickport CRM + GT06 Tracking  
**Environment:** Production  
**AWS Region:** ap-south-1  
**Criticality:** 🔴 EMERGENCY USE  
**Last Verified:** Feb 2026  

---

## 1. Purpose

This playbook restores **100% functionality** in case of:

- Accidental instance deletion
- OS corruption
- Disk failure
- Unrecoverable SSH lockout
- Human error

Goal:
> **Restore production within 30 minutes with zero data loss**

---

## 2. Prerequisites (Must Exist)

Before recovery, ensure:

- ✅ Latest EBS snapshot exists
- ✅ Elastic IP is NOT released
- ✅ AWS Console access available

If any of the above is missing → STOP and reassess.

---

## 3. Scenario A — Instance Deleted / Unbootable

### 3.1 Launch Recovery Instance

1. EC2 → Launch Instance
2. AMI: **Ubuntu 24.04 LTS**
3. Instance type: same or higher than original
4. **DO NOT attach storage yet**
5. Launch

---

### 3.2 Restore Root Volume from Snapshot

1. EC2 → Snapshots
2. Select correct snapshot
3. Create Volume
   - AZ must match new instance
4. Wait until volume is `available`

---

### 3.3 Attach Volume as Root

1. Stop recovery instance
2. Attach restored volume:
   - Device: `/dev/sda1`
3. Detach temporary root volume
4. Start instance

---

## 4. Attach Elastic IP (MANDATORY)

1. EC2 → Elastic IPs
2. Select existing Elastic IP
3. Associate with recovered instance

⚠ GT06 devices will reconnect **automatically**.

---

## 5. Post-Boot Verification (CRITICAL)

SSH into instance and verify:

### 5.1 Disk & Data
```bash
df -h
ls /var/www
ls /var/lib/mysql
5.2 Services
sudo systemctl status nginx
sudo systemctl status mysql
sudo systemctl status traccar
pm2 list
Expected:

All services = active

GT06 port = listening

sudo ss -lntp | grep 5023
5.3 Application Health
cd /var/www/quickport-laravel
php artisan migrate:status
No pending or failed migrations.

6. Scenario B — SSH Broken but Disk OK
6.1 Rescue Method
Stop production instance

Detach root volume

Attach to Rescue Instance

Mount volume:

sudo mount /dev/xvdf1 /mnt
6.2 Fix SSH
sudo nano /mnt/etc/ssh/sshd_config
Ensure:

PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
Check:

ls /mnt/home/ubuntu/.ssh/authorized_keys
Unmount, reattach to production instance, start.

7. Scenario C — Database Corruption Only
Preferred:

Restore snapshot

Extract /var/lib/mysql

Fallback:

gunzip < backup.sql.gz | mysql quickport_crm_laravel
⚠ Use SQL restore only if snapshot restore is impossible.

8. Final Validation Checklist
 Elastic IP reachable

 GT06 devices updating

 CRM login works

 PM2 services running

 Disk usage < 80%

9. Recovery Time Expectation
Step	Time
Snapshot restore	5–10 min
Instance launch	3–5 min
Validation	5–10 min
⏱ Total: ~20–30 minutes

10. Absolute Rules
❌ Never panic
❌ Never rebuild manually
❌ Never skip snapshot validation

Recovery is a process, not a guess.

End of Document


---
