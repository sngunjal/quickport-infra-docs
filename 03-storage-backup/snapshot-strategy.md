# Snapshot & Backup Strategy (EC2 + Database + GT06)

**Owner:** Quickport  
**Environment:** Production  
**Last Verified:** Feb 2026  

---

## 1. Purpose

This document defines a **non-negotiable snapshot and backup discipline** to guarantee:

- Zero data loss
- Fast recovery from any failure
- Confidence during maintenance
- Protection against human error

This strategy assumes **no dedicated DevOps team**.

---

## 2. What Must Be Protected (Priority Order)

| Priority | Asset | Reason |
|-------|------|-------|
| P0 | Root EBS Volume | OS + code + DB + configs |
| P0 | MariaDB Data | Financial + CRM data |
| P1 | GT06 Server Code | Live GPS ingestion |
| P1 | Laravel `.env` | Secrets & credentials |
| P2 | Logs | Diagnostics only |

---

## 3. Snapshot Types (Mandatory)

### 3.1 Manual Snapshots (Human-Controlled)

Used **before ANY risky operation**.

**Examples:**
- Disk cleanup
- Volume resize
- SSH fix
- OS update
- Service refactor

**Rules:**
- Must include date + reason
- Must reach `completed` state
- Must never be deleted manually

**Naming Convention:**
prod-root-YYYY-MM-DD-before-<action>


Example:
prod-root-2026-02-07-before-disk-resize


---

### 3.2 Automated Snapshots (Safety Net)

Configured via **AWS Backup or EventBridge**.

**Minimum:**
- Daily snapshot
- Retention: 7 days

⚠ Automated backups do **not** replace manual snapshots.

---

## 4. Snapshot Locking & Recycle Bin

### 4.1 Recycle Bin (Soft Protection)

- Enabled for EBS snapshots
- Retention: **7 days**

Purpose:
- Recover from accidental deletion
- Human error protection

---

### 4.2 Snapshot Lock (Hard Protection)

Use **only for critical baseline snapshots**.

| Mode | Use Case |
|---|---|
| Unlocked | Daily / routine snapshots |
| Locked | Pre-migration / financial milestone |

⚠ Locked snapshots:
- Cannot be deleted
- Cannot be modified
- Cost applies for full retention period

---

## 5. AMI vs Snapshot (Clear Rule)

| Situation | Use |
|------|----|
| Disk resize | Snapshot |
| SSH broken | Snapshot |
| OS upgrade | AMI |
| Kernel change | AMI |
| Architecture change | AMI |

**Never create AMI casually**.

---

## 6. Restore Scenarios (Step-by-Step)

---

### 6.1 Restore Entire Instance (Worst Case)

1. Stop instance
2. Detach root volume
3. Create volume from snapshot
4. Attach as `/dev/sda1`
5. Start instance
6. Reattach Elastic IP

⏱ Typical Recovery Time: **10–15 min**

---

### 6.2 Restore File or Directory Only

1. Create temp instance
2. Attach snapshot volume
3. Mount volume
4. Copy required files
5. Detach volume
6. Terminate temp instance

---

## 7. Database Backup (Additional Layer)

Snapshots protect DB, but **logical backups still required**.

### Manual DB backup
```bash
mysqldump -u root -p quickport_crm_laravel > /backups/db_$(date +%F).sql
Store:

/var/backups

Or S3 (recommended)

8. Verification Checklist (After Snapshot)
Snapshot state = completed

Correct volume ID

Correct instance ID

Elastic IP noted

Timestamp logged

9. Forbidden Practices
❌ Running maintenance without snapshot
❌ Deleting snapshots to save cost
❌ Relying only on automated backups
❌ Modifying production without restore plan

10. Cost Reality (India – Approx)
Item	Cost
30 GB snapshot	₹60–₹90 / month
Daily 7-day retention	₹15–₹25
AMI	Snapshot cost only
Snapshots are cheaper than downtime.

11. Snapshot Discipline Rule
If you hesitate whether a snapshot is needed → take one.

Snapshots are your undo button.

12. Status
Approved: ✅
Mandatory for production: ✅
Exception allowed: ❌


---

## 🔒 What this file protects you from

- Disk mistakes  
- Wrong volume attachment  
- SSH lockout  
- Accidental deletes  
- Human fatigue errors  

---
