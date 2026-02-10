# Permanent Protection Checklist (Production)

**System:** Quickport CRM + GT06 Tracking  
**Environment:** Production  
**AWS Region:** ap-south-1  
**Owner:** Quickport  
**Criticality:** 🔴 CRITICAL  
**Last Verified:** Feb 2026  

---

## 1. Objective

This checklist ensures **irreversible protection** of:

- Databases (MariaDB)
- Application code (Laravel, Node.js)
- GT06 tracking services
- AWS account access
- Human-error risks

If followed fully, **data loss becomes practically impossible**.

---

## 2. Identity & Access Management (IAM)

### 2.1 Root Account Protection (MANDATORY)

✅ Enable MFA on **AWS Root Account**  
✅ Store root credentials **offline only**  
❌ Never use root for daily work  

Checklist:
- [ ] MFA enabled
- [ ] Root password stored offline
- [ ] Root login tested once only

---

### 2.2 Admin IAM User

Create **one admin IAM user**.

Permissions:
- AdministratorAccess

Security:
- [ ] MFA enabled
- [ ] Strong password
- [ ] Access keys rotated yearly

❌ Never create multiple admins

---

## 3. SSH Access Protection (EC2)

### 3.1 SSH Key Rules

| Rule | Status |
|---|---|
| One production PEM only | ⬜ |
| PEM never emailed/shared | ⬜ |
| Backup PEM stored offline | ⬜ |
| No password SSH login | ⬜ |

---

### 3.2 SSH Hardening (Instance)

Check:
```bash
sudo nano /etc/ssh/sshd_config
Ensure:

PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
Apply:

sudo systemctl restart ssh
4. Instance-Level Safeguards
4.1 Termination Protection
Enable on production instance:

EC2 → Instance → Modify → Termination Protection → ENABLE
✅ Prevents accidental delete

4.2 Stop Protection (Recommended)
EC2 → Instance → Modify → Stop Protection → ENABLE
5. Storage Protection (EBS)
5.1 Volume Lock-in Rules
Rule	Description
Delete on Termination	❌ DISABLED
Volume resizing	Allowed
Volume detachment	Only during rescue
5.2 Snapshot Locking (Recycle Bin)
Retention Rule:

Type: EBS Snapshot

Retention: 30–90 days

Mode: Unlocked

✅ Prevents accidental snapshot deletion

6. Database Protection (MariaDB)
6.1 Local Backup (Daily)
Cron (recommended):

0 2 * * * mysqldump quickport_crm_laravel | gzip > /var/backups/db_$(date +\%F).sql.gz
6.2 Snapshot Dependency
⚠ Database backups alone are NOT enough
✅ EBS snapshots are authoritative restore source

7. Application Code Protection
7.1 Git Discipline
Rules:

 GitHub remote exists

 main protected

 No direct production-only code

Never trust:
❌ Only-server copy
❌ Manual edits without git

7.2 Environment Files
Files to protect:

.env
.env.production
Rules:

❌ Never commit

✅ Backed up via snapshot

✅ Permissions: 640

8. Service Protection
8.1 PM2 Services
Save state after every change:

pm2 save
Enable auto-start:

pm2 startup
8.2 Systemd Services
Ensure enabled:

sudo systemctl enable traccar
sudo systemctl enable nginx
sudo systemctl enable mysql
9. Network Protection
9.1 Elastic IP (MANDATORY)
Rules:

One Elastic IP

Attached to production only

Never released

GT06 depends on this.

10. Human Error Controls
10.1 Before ANY Change Checklist
Before touching production:

 Snapshot exists

 Git pushed

 Services status checked

 Disk space checked

 Elastic IP attached

10.2 Forbidden Actions
❌ Delete production instance
❌ Resize volume without snapshot
❌ Restart without checking GT06 port
❌ Test commands on production blindly

11. Disaster Recovery Confidence
If instance is lost:

Launch new EC2

Attach volume snapshot

Attach Elastic IP

Start services

GT06 resumes automatically

⏱ Expected recovery time: < 30 minutes

12. Sign-off
This checklist is mandatory for:

Maintenance

Scaling

Debugging

Emergency recovery

Deviation = Risk

End of Document


---
