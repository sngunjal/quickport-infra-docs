# GT06 GPS Service

## Overview
Custom TCP server listening on port `5023`
for GT06 GPS devices.

## Startup
```bash
pm2 start index.js --name gt06-gps-service
pm2 save

# GT06 GPS Service

## Overview
Custom TCP server listening on port `5023`
for GT06 GPS devices.

## Startup
```bash
pm2 start index.js --name gt06-gps-service
pm2 save
Health Check
sudo ss -lntp | grep 5023
Common Issues
Port Conflict
Cause: Another process binding to 5023

Fix:

sudo ss -lntp | grep 5023
sudo pkill node
sudo systemctl restart traccar
Device Configuration
IP: Elastic IP only

Port: 5023


---

## `07-failover/disaster-recovery.md`

```md
# Disaster Recovery Plan

## Scenarios Covered
- SSH lockout
- Disk full
- Instance corruption
- Accidental deletion

---

## SSH Recovery Steps

1. Stop instance
2. Detach root volume
3. Attach to rescue instance
4. Fix filesystem / SSH
5. Reattach and boot

Snapshots guarantee data safety.
