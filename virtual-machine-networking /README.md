# Azure VM Lab: Virtual Machine + Networking + Monitoring

## 🎯 Purpose
Build a Linux VM in Azure, secure it with networking (VNet, Subnet, NSG allowing only SSH), and enable Azure Monitor (metrics + alerts).  
This project demonstrates **core Azure services** (compute, networking, monitoring) along with **security and SLA/HA concepts**.  

---

## 📂 Files / Screenshots
- `/screenshots/rg-create.png` — Resource Group overview  
- `/screenshots/vnet-subnet.png` — VNet + Subnet config  
- `/screenshots/nsg-config.png` — NSG inbound rules  
- `/screenshots/pip-create.png` — Public IP overview  
- `/screenshots/nic-create.png` — NIC overview  
- `/screenshots/vm-setup.png` — VM creation summary  
- `/monitoring/insights-law.png` — Log Analytics Workspace + VM Insights  
- `/monitoring/dcr-setup.png` — Data Collection Rule overview  
- `/monitoring/metrics-dashboard.png` — CPU usage chart  

---

## 🚀 Portal Steps (Summary)

1. **Create Resource Group** → `rg-vm-lab`  
   ![Resource Group](/screenshots/rg-create.png)  

2. **Create VNet + Subnet** → `vnet-vm-lab` + `subnet-01`  
   ![VNet + Subnet](/screenshots/vnet-subnet.png)
   ![VNet + Subnet](/screenshots/vnet.png) 

4. **Create NSG** → `nsg-vm-lab`  
   - Add inbound rule `Allow-SSH-From-MyIP` (priority `100`, source `<YOUR_IP>/32`, dest port `22`, protocol `TCP`, action `Allow`)  
   ![NSG Rules](/screenshots/nsg-config.png)  

5. **Create Public IP** → `pip-vm-lab` (Static, Standard)  
   ![Public IP](/screenshots/pip-create.png)  

6. **Create NIC** → `nic-vm-lab` attaching NSG + PIP  
   ![NIC](/screenshots/nic-create.png)  

7. **Create VM** → `vm-linux-lab` (Ubuntu, SSH key auth)  
   ![VM Setup](/screenshots/vm-setup.png)  

8. **Test SSH connection**  
   ```bash
   ssh azureuser@<VM_Public_IP>
9. Portal → VM → Monitoring → Insights → Enable (select `law-vm-lab`)
10. Create Action Group `ag-vm-lab` (email)
11. Create Alert rule (e.g., CPU >80% for 5min) and attach `ag-vm-lab`

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
