# Backup & Snapshot Discipline (Production)

**System:** Quickport CRM + GT06 Tracking  
**Environment:** Production  
**AWS Region:** ap-south-1  
**Applies To:** EC2, EBS, MariaDB, Application Code  
**Criticality:** 🔴 CRITICAL  
**Last Verified:** Feb 2026  

---

## 1. Purpose

This document defines a **non-negotiable backup strategy** that guarantees:

- Zero permanent data loss
- Fast recovery from human error
- Protection against AWS / OS / disk failure
- Safe maintenance without fear

If followed correctly, **production data cannot be lost**.

---

## 2. Backup Layers (Defense in Depth)

| Layer | What It Protects | Mandatory |
|---|---|---|
| EBS Snapshot | Entire system (OS + DB + Code) | ✅ YES |
| Database Dump | Logical DB restore | ✅ YES |
| Git Repository | Source code history | ✅ YES |
| AMI | Full machine template | 🔶 Recommended |

---

## 3. EBS Snapshot Strategy (PRIMARY)

### 3.1 Why Snapshots Are Primary

EBS snapshots capture:
- `/var/www`
- `/var/lib/mysql`
- `/home/ubuntu`
- System configuration
- Hidden metadata

⚠ **Snapshots are the single source of truth**

---

### 3.2 Snapshot Frequency

| Type | Frequency | Retention |
|---|---|---|
| Manual (Before changes) | Every maintenance | Until verified |
| Automated | Daily | 30–90 days |
| Emergency | Before risky ops | Until stable |

---

### 3.3 Snapshot Creation (Manual)

**AWS Console → EC2 → Volumes → Select Volume → Create Snapshot**

Required fields:
- Description:
PROD_BEFORE_<CHANGE>_<YYYY-MM-DD>


Example:
PROD_BEFORE_VOLUME_RESIZE_2026-02-07


---

### 3.4 Snapshot Protection (Recycle Bin)

Rule:
- Resource: **EBS Snapshots**
- Retention: **30 or 90 days**
- Mode: **Unlocked**

✅ Prevents accidental deletion  
❌ Does not prevent intentional cleanup

---

## 4. Database Backups (SECONDARY)

### 4.1 Why DB Dumps Still Matter

Snapshots restore **everything**  
DB dumps restore **only data**

Used for:
- Table-level restore
- Corruption debugging
- Partial recovery

---

### 4.2 Manual Database Backup

```bash
mysqldump quickport_crm_laravel \
  | gzip > /var/backups/quickport_db_$(date +%F).sql.gz
Verify:

ls -lh /var/backups
4.3 Automated Daily DB Backup (Cron)
sudo crontab -e
Add:

0 2 * * * mysqldump quickport_crm_laravel | gzip > /var/backups/db_$(date +\%F).sql.gz
Retention:

find /var/backups -type f -mtime +14 -delete
5. Git-Based Code Backup
5.1 Mandatory Git Rules
All production code must exist in GitHub

No server-only changes

.env never committed

Checklist:

 git status clean

 git push done before maintenance

 Tags before major changes

5.2 Pre-Maintenance Git Tag
git tag prod-pre-maintenance-2026-02-07
git push origin --tags
6. AMI Strategy (SYSTEM TEMPLATE)
6.1 When to Create AMI
Create AMI:

After major stabilization

Before OS upgrade

Before disk restructuring

AMI captures:

Instance config

Boot configuration

Base services

6.2 AMI Creation Rules
Options:

❌ Reboot instance → NO (unless kernel work)

✅ Tag image and snapshots together

Tags:

Name: quickport-prod-ami

Environment: production

Date: YYYY-MM-DD

7. Restore Procedures (Verified)
7.1 Full System Restore (Worst Case)
Launch EC2

Attach snapshot → new volume

Attach Elastic IP

Start instance

Validate services

⏱ Recovery: 15–30 minutes

7.2 Database-Only Restore
gunzip < db_YYYY-MM-DD.sql.gz | mysql quickport_crm_laravel
8. Verification Checklist
After any backup:

 Snapshot status = completed

 DB dump file exists

 Disk space OK

 No running write jobs

9. Forbidden Backup Mistakes
❌ Relying only on DB dumps
❌ No snapshot before maintenance
❌ Manual copy-paste backups
❌ Deleting snapshots to save cost

10. Cost vs Safety Reality
Item	Monthly Cost	Risk Reduction
Snapshots	Low	Massive
Extra 20GB	Minor	Huge
Elastic IP	₹0 (attached)	Critical
Never optimize cost at the expense of backups.

11. Ownership & Enforcement
This discipline applies to:

You

Future admins

Emergency recovery

No exceptions.

End of Document


---
