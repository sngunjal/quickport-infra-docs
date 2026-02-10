# Elastic IP & Network Stability (Production)

**System:** Quickport CRM + GT06 Tracking  
**AWS Service:** EC2, Elastic IP, Security Groups  
**Environment:** Production  
**Owner:** Quickport  
**Last Verified:** Feb 2026  

---

## 1. Purpose

This document defines the **permanent networking contract** for:

- GT06 GPS devices
- Node.js TCP server (Port 5023)
- Web services (HTTP/HTTPS)
- Zero reconnection failures after restart / crash

Once followed, **device IP will NEVER change**.

---

## 2. Problem Statement (Observed)

Without Elastic IP:
- EC2 public IP changes on:
  - Stop / Start
  - Crash recovery
  - Instance replacement
- GT06 devices:
  - Keep sending data to old IP
  - Appear “offline”
  - Cause silent data loss

⚠ This already occurred once and must never repeat.

---

## 3. Solution (Non-Negotiable)

✅ **Elastic IP (EIP)** must be attached to the **production instance only**.

This guarantees:
- Fixed IPv4 address
- Device stability
- Predictable firewall rules
- Safe maintenance

---

## 4. Current Production Instance

| Item | Value |
|---|---|
| Instance ID | `i-0ae922662c241c912` |
| Role | Production (CRM + GT06) |
| Volume | 30 GB |
| Region | ap-south-1 |

---

## 5. Elastic IP Strategy

### 5.1 Reuse Existing Elastic IP (Recommended)

Elastic IP:
- **3.109.42.82**
- Currently attached to stopped instance:
  - `i-03a38bdaf466d4d23`

✅ This IP is already trusted by devices  
✅ No tracker reconfiguration needed  

---

### 5.2 Safe Reassignment Steps

#### Step 1 — Ensure old instance is stopped
```text
EC2 → Instances → i-03a38bdaf466d4d23 → State: STOPPED
Step 2 — Disassociate Elastic IP
EC2 → Elastic IPs → 3.109.42.82
Actions → Disassociate Elastic IP
Step 3 — Associate to Production Instance
Actions → Associate Elastic IP
Instance: i-0ae922662c241c912
Step 4 — Verify from Instance
curl ifconfig.me
Expected output:

3.109.42.82
6. GT06 Device Configuration (Final)
Parameter	Value
Server IP	3.109.42.82
Port	5023
Protocol	TCP
Keepalive	Enabled
⚠ Never point devices to non-Elastic IPs again.

7. Security Group Rules (Mandatory)
Inbound Rules
Type	Port	Source
Custom TCP	5023	0.0.0.0/0 (or device IP range)
HTTP	80	0.0.0.0/0
HTTPS	443	0.0.0.0/0
SSH	22	Admin IP only
8. Billing Clarification (Elastic IP)
Condition	Cost
EIP attached to running instance	₹0
EIP unattached	Charged
Multiple unused EIPs	Charged
✅ Always keep one EIP attached
❌ Never leave unused EIPs idle

9. Forbidden Actions
❌ Using public dynamic IP for devices
❌ Stopping production without checking EIP
❌ Deleting EIP accidentally
❌ Sharing EIP across environments

10. Verification Checklist
Check	Status
Elastic IP attached	⬜
Instance restarted safely	⬜
GT06 devices reporting	⬜
Port 5023 listening	⬜
Command:

sudo ss -lntp | grep 5023
Expected:

LISTEN ... java/node ... :5023
11. Outcome
After this:

GT06 never disconnects due to IP change

Maintenance becomes safe

No panic during restart

Predictable system behavior

End of Document


---
