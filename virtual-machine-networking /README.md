# azure-vm-lab

## Week 2 — Virtual Machine + Networking + Monitoring

### Purpose
Create a Linux VM, secure networking (VNet, Subnet, NSG allowing only SSH), and enable Azure Monitor (metrics + alerts).

### Files / Screenshots
- `/screenshots/vm-setup.png` — VM creation summary
- `/monitoring/alert-config.png` — Alert rule UI

---

## Portal steps (summary)
1. Create resource group `rg-vm-lab`
2. Create VNet `vnet-vm-lab` (10.0.0.0/16) with subnet `subnet-01` (10.0.1.0/24)
3. Create NSG `nsg-vm-lab`
   - Add inbound rule `Allow-SSH-From-MyIP`:
     - Priority `100`, Source `<YOUR_IP>/32`, Destination port `22`, Protocol `TCP`, Action `Allow`
4. Create Public IP `pip-vm-lab` (Static, Standard)
5. Create NIC `nic-vm-lab` attaching `nsg-vm-lab` and `pip-vm-lab`
6. Create VM `vm-linux-lab` (Ubuntu), attach NIC or let Portal create. Use SSH key for auth.
7. Create Log Analytics workspace `law-vm-lab`
8. Portal → VM → Monitoring → Insights → Enable (select `law-vm-lab`)
9. Create Action Group `ag-vm-lab` (email)
10. Create Alert rule (e.g., CPU >80% for 5min) and attach `ag-vm-lab`

---

## CLI reference
(see CLI commands in the repo `scripts/` for `az group create`, `az network vnet create`, `az network nsg create`, `az vm create`, etc.)

---

## NSG configuration (secure networking)
| Name                | Priority | Source        | Dest Port | Protocol | Action |
|---------------------|----------|---------------|-----------|----------|--------|
| Allow-SSH-From-MyIP | 100      | `<YOUR_IP>/32`| 22        | TCP      | Allow  |
| Default rules       | 65000+   | VirtualNetwork| *         | Any      | Allow/Deny (defaults) |

> Only SSH is allowed inbound from my IP. Outbound uses default allow.

---

## Monitoring & Alerts
- Log Analytics workspace: `law-vm-lab`  
- VM Insights enabled → collects CPU/Disk/Network metrics and process info.  
- Alert created: `VM-CPU-High` (CPU >80% for 5m) → Action Group `ag-vm-lab` sends email.

---

## SLA: Single VM vs Availability Zone (summary)
- **Single VM (single instance)**: single point of failure — VM or underlying host restart impacts availability. Use for dev/test or when low availability is acceptable.
- **Availability Set**: multiple VMs across fault domains and update domains — protects against host hardware failures and planned maintenance.
- **Availability Zones**: VMs deployed to different physical datacenter zones in the same region — provides higher resilience to datacenter outages/zone failures.

> Note: Azure provides different SLA guarantees depending on whether you run a single instance, multiple instances in an availability set, or across availability zones. Please check the official Azure SLA page for the exact numeric guarantees for your chosen VM SKU/region.

---

## Cleanup
To avoid charges: delete `rg-vm-lab` when finished:
```bash
az group delete --name rg-vm-lab --yes --no-wait
