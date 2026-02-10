# EBS Volumes — Protection, Expansion & Recovery Strategy

**System:** Quickport CRM + GT06 Tracking  
**Environment:** Production  
**AWS Region:** ap-south-1  
**Owner:** Quickport  
**Criticality:** 🔴 CRITICAL  
**Last Verified:** Feb 2026  

---

## 1. Purpose

This document defines **authoritative rules** for managing EBS volumes so that:

- Application code is never lost
- Database data is never corrupted
- Disk expansion is safe and reversible
- Rescue operations do not cause accidental damage
- Billing surprises are avoided

This document applies to **ALL production volumes**.

---

## 2. Production Volume Identification

### 2.1 Authoritative Root Volume

| Item | Value |
|---|---|
| Instance ID | i-0ae922662c241c912 |
| Volume ID | vol-07011436804334588 |
| Device | /dev/xvda |
| Mount | / |
| Type | gp3 |
| Size | 30 GB |

⚠ **This volume contains EVERYTHING**
- Laravel code
- Node services
- GT06 server
- MariaDB databases
- Configuration files

---

## 3. Mandatory Volume Rules (NON-NEGOTIABLE)

### 3.1 Delete Protection

| Setting | Required |
|---|---|
| Delete on Termination | ❌ DISABLED |
| Detach allowed | ⚠ Only during rescue |
| Resize allowed | ✅ Yes (snapshot first) |

> ❌ If Delete-on-Termination is enabled → **DATA LOSS RISK**

---

### 3.2 One Volume Policy

- One root EBS volume
- No temporary “test” volumes attached to production
- No mixing of production & rescue data

---

## 4. Volume Expansion SOP (SAFE METHOD)

### 4.1 Before Resizing (MANDATORY)

- [ ] Instance running or stopped (either is fine)
- [ ] **Fresh snapshot created**
- [ ] Snapshot verified as completed
- [ ] No disk-intensive jobs running

❌ Never resize without snapshot.

---

### 4.2 AWS Console Steps

1. EC2 → Volumes
2. Select production volume
3. Modify → Change size
4. Apply

⚠ **No reboot required at AWS level**

---

### 4.3 OS-Level Resize (REQUIRED)

After AWS resize:

```bash
lsblk
Confirm new size is visible.

Then:

sudo growpart /dev/xvda 1
sudo resize2fs /dev/xvda1
Verify:

df -h /
Expected:

Increased available space

No data loss

5. Billing Clarity (NO SURPRISES)
5.1 EBS Pricing Reality (ap-south-1)
Size	Approx Monthly Cost
8 GB	Very low
30 GB	Still low (₹200–₹300 range)
Disk is cheaper than downtime

5.2 Snapshots Billing
Charged only for used blocks

Incremental (not full copy each time)

Locked snapshots may cost slightly more — worth it

6. Rescue & Recovery Volume Handling
6.1 When Rescue Instance Is Used
Allowed:

Detach volume

Attach to rescue

Mount read-only or read-write

Forbidden:

Formatting

Repartitioning

fsck without snapshot

6.2 After Rescue
Checklist:

 Volume reattached to production instance

 Rescue instance stopped or terminated

 Elastic IP attached back to production

 Services verified

7. Snapshot Relationship (Source of Truth)
Asset	Authority
Live volume	Runtime
Snapshot	Recovery source
DB dump	Secondary
Git	Code history only
Snapshots > DB dumps > Git

8. Verification Commands (Production)
Disk health:
df -h
lsblk
Volume mount:
mount | grep xvda
Space pressure alert:
df -h / | awk '$5+0 > 85 {print "WARNING: Disk usage high"}'
9. Forbidden Actions (ABSOLUTE)
❌ Delete production volume
❌ Recreate volume without snapshot
❌ Attach unknown volumes
❌ Format during panic
❌ Resize without OS-level expansion

One mistake = permanent loss.

10. Disaster Recovery Confidence
If volume is lost or corrupted:

Create volume from latest snapshot

Attach to new instance

Attach Elastic IP

Start services

⏱ Recovery time: 20–30 minutes

11. Sign-off
This document must be followed during:

Maintenance

Disk expansion

SSH rescue

Instance replacement

Emergency recovery

Deviation requires explicit risk acceptance.

End of Document


---

## ✅ What you now have (complete set)

| Area | Status |
|---|---|
| Snapshot strategy | ✅ |
| Database backups | ✅ |
| **EBS volume rules** | ✅ (just added) |
| Elastic IP strategy | ✅ |
| Zero-downtime SOP | ✅ |
| Permanent protection checklist | ✅ |

---
