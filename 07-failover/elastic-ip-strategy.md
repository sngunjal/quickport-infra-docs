# Elastic IP Strategy — GT06 GPS & Quickport CRM

**Owner:** Quickport  
**Applies To:** EC2 Instance `i-0ae922662c241c912`  
**Critical Services:**  
- GT06 GPS TCP Server (Port 5023)  
- Quickport CRM (Laravel + Nginx)  
- Supporting APIs (PM2-managed Node services)

---

## 1. Purpose

This document defines the **mandatory Elastic IP (EIP) policy** for Quickport
to prevent service outages caused by **public IP changes**, especially for:

- GT06 GPS devices (hard-coded IP + Port)
- External integrations
- Field-deployed trackers without DNS capability

---

## 2. Incident Background (Why This Exists)

### What happened
- EC2 instance was **stopped / started** during maintenance
- AWS assigned a **new public IP**
- GT06 devices continued sending data to the **old IP**
- Result:
  - Devices appeared “offline”
  - No location updates
  - False assumption of service failure

### Root cause
GT06 devices:
- Do **NOT** support DNS
- Require **fixed IP + port**
- Cannot auto-discover new server IPs

### Resolution
- Devices manually reconfigured to new IP
- Service restored
- **Permanent fix identified:** Elastic IP

---

## 3. Decision: Elastic IP Is Mandatory

### Rule (Non-Negotiable)

> **Any EC2 instance running GT06 GPS services MUST have an Elastic IP attached.**

Dynamic public IPs are **not acceptable** for production.

---

## 4. Elastic IP Reuse Policy

### Existing Elastic IP
- **Elastic IP:** `3.109.42.82`
- Previously associated with stopped instance `i-03a38bdaf466d4d23`

### Approved Action
- Detach Elastic IP from stopped instance
- Attach **same Elastic IP** to:
  - `i-0ae922662c241c912`

### Why reuse?
- Devices already configured
- No field reconfiguration needed
- Zero downtime for trackers

---

## 5. Correct Elastic IP Attachment Procedure

### Step 1: Ensure target instance is RUNNING
Elastic IP can be attached without stopping the instance.

### Step 2: Detach from old instance
AWS Console → EC2 → Elastic IPs → Actions → Disassociate

### Step 3: Associate to production instance
- Instance: `i-0ae922662c241c912`
- Network Interface: Primary
- Private IP: Auto-select

### Step 4: Verify
```bash
curl ifconfig.me
Expected output:

3.109.42.82
6. Post-Attachment Verification Checklist
Network
sudo ss -lntp | grep 5023
Expected:

LISTEN ... java/node ... :5023
GT06 Service
pm2 list
Expected:

gt06-gps-service → online

No port conflicts

Device Confirmation
At least one GT06 device updates location

Timestamp changes in UI

No reconnect storms in logs

7. Cost Impact (India – Mumbai Region)
Item	Cost
Elastic IP (attached, in use)	₹0
Elastic IP (detached)	~₹0.40/hour
Data transfer	Standard EC2 rates
Rule: Never leave an Elastic IP unattached.

8. Protection Rules
DO
Keep Elastic IP attached permanently

Document Elastic IP in infra docs

Use Elastic IP before onboarding devices

DO NOT
Stop instance assuming IP will remain same

Rely on dynamic IP for GT06

Terminate instance without checking EIP association

9. Disaster Recovery Note
If instance must be replaced:

Launch new EC2

Attach existing Elastic IP

Start GT06 + CRM services

Devices reconnect automatically

No field action required.

10. Status
Current State:
✅ Elastic IP strategy approved
⚠ Ensure Elastic IP is attached immediately if not already done

This document is binding for all future maintenance.


---

## ✅ What this gives you

- Permanent fix for **GT06 outages**
- Zero field reconfiguration
- Safe stop/start of EC2
- Predictable operations
- Audit-ready documentation

---
