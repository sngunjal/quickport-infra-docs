# Elastic IP & Networking Strategy (Production)

**Owner:** Quickport  
**Environment:** Production (EC2)  
**Last Verified:** Feb 2026  

---

## 1. Purpose

This document defines **network stability rules** for production servers to ensure:

- GPS devices (GT06) never disconnect due to IP change
- APIs and callbacks remain reachable
- No emergency reconfiguration after reboot
- Predictable operations during maintenance

---

## 2. Core Rule (Non-Negotiable)

> **Every production EC2 instance MUST have an Elastic IP attached.**

Dynamic public IPs are **not acceptable** for:
- GPS devices
- Webhooks
- Mobile apps
- Third-party callbacks

---

## 3. Why Elastic IP Is Mandatory

### What went wrong without it (real incident)

- Instance reboot → public IP changed
- GT06 devices kept sending data to old IP
- Devices appeared “offline”
- Manual reconfiguration required
- Live tracking broke during maintenance

Elastic IP **eliminates this entire class of failure**.

---

## 4. Elastic IP Basics (AWS)

| Item | Fact |
|---|---|
| Elastic IP | Static IPv4 address |
| Region-locked | Yes |
| Cost when attached | Free |
| Cost when unattached | ~$0.005/hour |

---

## 5. Attachment Strategy

### 5.1 One Elastic IP per Production Instance

| Instance Role | Elastic IP |
|---|---|
| CRM + API + GT06 | REQUIRED |
| Rescue / Temporary | OPTIONAL |
| Test / Sandbox | NOT REQUIRED |

---

### 5.2 Reusing an Existing Elastic IP (Recommended)

If a known Elastic IP already exists:

- Detach from old instance (stopped)
- Attach to current production instance

This preserves:
- Device configs
- DNS records
- Client trust

---

## 6. Safe Elastic IP Reassignment Procedure

### Step-by-step (Zero Risk)

1. **Stop old instance**
2. Detach Elastic IP
3. Attach Elastic IP to new instance
4. Start new instance
5. Verify:
   - SSH access
   - API access
   - GT06 connectivity

⏱ Downtime: **< 1 minute**

---

## 7. GT06-Specific Rules

### 7.1 Device Configuration (Permanent)

| Setting | Value |
|---|---|
| IP | Elastic IP only |
| Port | 5023 |
| Protocol | TCP |
| Retry | Enabled |

❌ Never configure GT06 with a dynamic IP.

---

### 7.2 Verification Command

On server:
```bash
sudo ss -lntp | grep 5023
Expected:

LISTEN *:5023 users:(("node" or "java"))
8. Firewall & Security Group Rules
Inbound Rules (Minimum)
Port	Source	Purpose
22	Your IP only	SSH
80	0.0.0.0/0	HTTP
443	0.0.0.0/0	HTTPS
5023	0.0.0.0/0	GT06 GPS
8082	INTERNAL / LIMITED	Traccar (if used)
⚠ Never open SSH to 0.0.0.0/0.

9. DNS Strategy (Optional but Recommended)
If using domain:

gps.quickport.co.in → Elastic IP
api.quickport.co.in → Elastic IP
crm.quickport.co.in → Elastic IP
DNS + Elastic IP = maximum flexibility

10. Elastic IP Audit Checklist
Run after every maintenance:

 Elastic IP attached

 Instance state = running

 GT06 devices reporting

 API reachable

 SSH access confirmed

11. Forbidden Practices
❌ Production without Elastic IP
❌ Rebooting before checking EIP attachment
❌ Configuring devices with dynamic IP
❌ Leaving Elastic IP unattached (bill leakage)

12. Incident Recovery Rule
If GT06 devices stop reporting:

Check Elastic IP attached?

Check port 5023 listening?

Check Security Group rules

Check pm2 / traccar status

Do not touch device configs first.

13. Status
Mandatory: ✅
Reviewed after incident: ✅
Exception allowed: ❌

Elastic IP is not an “extra”.
It is part of your production identity.


---

## ✅ You now have:

- Storage safety (`snapshot-strategy.md`)
- Network stability (`elastic-ip-and-networking.md`)
- Zero-downtime SOP already created earlier

---
