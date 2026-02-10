# Elastic IP Governance (Production)

**System:** Quickport CRM + GT06 Tracking  
**Environment:** Production  
**AWS Region:** ap-south-1  
**Criticality:** 🔴 CRITICAL  
**Last Verified:** Feb 2026  

---

## 1. Purpose

Elastic IP (EIP) is a **hard dependency** for:

- GT06 GPS devices
- External device firmware configuration
- Long-lived TCP connections

Loss or reassignment of EIP = **fleet outage**.

---

## 2. Golden Rules (NON-NEGOTIABLE)

| Rule | Status |
|---|---|
| Only ONE Elastic IP | Mandatory |
| Elastic IP NEVER released | Mandatory |
| Elastic IP attached ONLY to production | Mandatory |
| Rescue instances NEVER get Elastic IP | Mandatory |

---

## 3. Production Elastic IP

| Item | Value |
|---|---|
| Elastic IP | 3.109.42.82 |
| Allocation ID | eipalloc-xxxxxxxx |
| Production Instance ID | i-0ae922662c241c912 |
| GT06 Port | 5023 |

---

## 4. Allowed Operations

✅ Reassociate Elastic IP **only when**:
- Production instance is restarted
- Instance is replaced from snapshot

❌ NOT allowed:
- Release Elastic IP
- Attach to rescue instance
- Attach to test instance

---

## 5. Recovery Procedure (Approved)

If production instance is lost:

1. Launch new EC2 from latest AMI or snapshot
2. Attach restored EBS volume
3. **Reattach Elastic IP**
4. Start services:
   ```bash
   sudo systemctl start nginx mysql
   pm2 resurrect
   sudo systemctl start traccar
Verify:

ss -lntp | grep 5023
GT06 devices reconnect automatically.

6. Violation Impact
Violation	Impact
Elastic IP released	🔴 Total outage
Wrong attachment	🔴 Fleet offline
IP changed	🔴 Devices unreachable
7. Final Note
Elastic IP is infrastructure identity, not networking convenience.

Treat it like a serial number, not a resource.

End of Document


---

