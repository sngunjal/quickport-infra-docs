# PM2 & Port Ownership Contract

**System:** Quickport CRM + GT06  
**Environment:** Production  
**Last Verified:** Feb 2026  

---

## 1. Objective

Prevent **silent service conflicts**, especially for GT06 TCP ingestion.

This document defines **exclusive ownership** of ports.

---

## 2. Port Ownership Table (AUTHORITATIVE)

| Port | Protocol | Owner | Service |
|---|---|---|---|
| 5023 | TCP | PM2 | gt06-gps-service |
| 8082 | HTTP | systemd | Traccar (web/admin only) |
| 80 | HTTP | systemd | Nginx |
| 443 | HTTPS | systemd | Nginx |
| 3306 | TCP | systemd | MariaDB |

---

## 3. GT06 Rule (CRITICAL)

> **Only ONE process may bind port 5023.**

Owner:
```text
PM2 → gt06-gps-service → node index.js
❌ Traccar must NEVER bind 5023
❌ Any other Node service must NEVER bind 5023

4. PM2 Canonical Services
Approved PM2 list:

pm2 list
Expected:

gt06-gps-service

quickport-crm-api

5. Conflict Detection Command
Before restarting GT06:

sudo ss -lntp | grep 5023
Expected output:

users:(("node",pid=XXXX))
❌ If Java owns it → STOP Traccar first

6. Recovery from Conflict
pm2 stop gt06-gps-service
sudo systemctl stop traccar
pm2 start gt06-gps-service
Then verify:

ss -lntp | grep 5023
7. Persistence Rules
After ANY PM2 change:

pm2 save
Ensure PM2 auto-start:

pm2 startup
8. Forbidden Actions
❌ Bind GT06 to dynamic port
❌ Run GT06 manually without PM2
❌ Allow Traccar to ingest GT06

End of Document


---
