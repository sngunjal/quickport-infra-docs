# Zero-Downtime Maintenance SOP

## Purpose
Perform maintenance without breaking:
- CRM access
- GT06 device connectivity
- API services

---

## Pre-Maintenance Checklist

- [ ] Elastic IP attached
- [ ] Manual EBS snapshot completed
- [ ] Disk usage < 80%
- [ ] GT06 port 5023 ownership verified

```bash
df -h
sudo ss -lntp | grep 5023
pm2 list

Maintenance Order

MySQL

Backend services

PM2 apps

Nginx

sudo systemctl restart mysql
pm2 restart all
sudo systemctl restart nginx

Post-Maintenance Validation

 CRM reachable

 GT06 devices sending data

 No port conflicts

sudo ss -lntp | grep 5023
pm2 logs gt06-gps-service --lines 50


---

## `05-maintenance/monthly-checklist.md`

```md
# Monthly Maintenance Checklist

## Infrastructure
- [ ] Disk usage check
- [ ] Snapshot verification
- [ ] Elastic IP attached

## Security
- [ ] SSH keys reviewed
- [ ] Security groups audited

## Services
- [ ] PM2 processes stable
- [ ] GT06 connectivity tested

## Backups
- [ ] Database backups present
- [ ] Snapshot retention correct
