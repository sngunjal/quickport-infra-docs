# GT06 GPS Service Architecture (Production)

**Service:** GT06 TCP GPS Ingestion  
**Owner:** Quickport  
**Environment:** Production  
**Port:** 5023  
**Last Verified:** Feb 2026  

---

## 1. Purpose

This document defines the **authoritative architecture** for GT06 GPS device handling.

Goals:
- Prevent port conflicts
- Avoid duplicate listeners
- Guarantee device connectivity after reboot
- Make GT06 service restart-safe and maintenance-safe

---

## 2. High-Level Architecture

```text
GT06 Device
   |
   |  TCP (5023)
   v
Node.js GT06 Server (PM2)
   |
   |  HTTP / Internal API
   v
Quickport Backend / Database
   |
   v
Custom UI (Maps / Dashboard)
3. Core Rule (Absolute)
ONLY ONE service may listen on TCP port 5023.

Violations of this rule will:

Block GT06 packets

Cause silent device drop

Break live tracking

4. Port Ownership Table
Port	Owner	Status
5023	GT06 Node.js Service	✅ EXCLUSIVE
5055	❌ UNUSED	BLOCKED
8082	Traccar Web (optional)	LIMITED
80/443	Nginx	STANDARD
⚠ Traccar must not bind to 5023 if using custom GT06 service.

5. GT06 Service Implementation
5.1 Service Location
/home/ubuntu/gt06-server/
├── index.js
├── package.json
└── logs/
5.2 Manual Test (Baseline)
cd ~/gt06-server
node index.js
Expected output:

✅ Listening on TCP port 5023 (GT06 Ready)
If this fails → do not proceed further.

6. PM2 Process Management (Mandatory)
6.1 Start Service
pm2 start index.js --name "gt06-gps-service"
6.2 Save Process List
pm2 save
6.3 Enable Startup on Boot
pm2 startup
# then run the command PM2 prints
7. PM2 Validation Checklist
pm2 list
Expected:

gt06-gps-service → online

No duplicate GT06 services

No zombie Node processes

8. Port Validation (Mandatory After Any Change)
sudo ss -lntp | grep 5023
Expected:

LISTEN *:5023 users:(("node",pid=XXXX))
❌ If java appears → Traccar conflict
❌ If multiple listeners → misconfiguration

9. Traccar Compatibility Rule
Allowed:
Traccar Web UI

Traccar DB migrations

Traccar admin tools

Forbidden:
Traccar binding to port 5023

Traccar auto-starting GT06 protocol

If Traccar is installed:

sudo systemctl stop traccar
sudo systemctl disable traccar
(unless explicitly required)

10. GT06 Device Configuration (Final)
Setting	Value
IP	Elastic IP only
Port	5023
Protocol	TCP
APN	As per SIM
Retry	Enabled
⚠ Never use:

Domain without Elastic IP

Dynamic EC2 IP

Port other than 5023

11. Health Check SOP
Server-side
pm2 status gt06-gps-service
sudo ss -lntp | grep 5023
Device-side
Location updates visible

No “offline” flicker

Stable heartbeat

12. Failure Recovery SOP
If GT06 devices stop reporting:

Check Elastic IP attached

Check port 5023 listener

Check PM2 process

Restart GT06 service:

pm2 restart gt06-gps-service
Do NOT touch device config unless step 1 fails

13. Forbidden Actions
❌ Running GT06 via node index.js in background
❌ Running multiple Node GT06 servers
❌ Mixing GT06 with Traccar port
❌ Rebooting without pm2 save

14. Status
Architecture locked: ✅

Incident resolved: ✅

Future-proofed: ✅

GT06 stability is architecture, not luck.
This document is your protection.


---

## ✅ What you now have documented

- Elastic IP stability
- GT06 port ownership
- PM2 lifecycle rules
- Traccar conflict prevention
- Incident recovery logic

---
