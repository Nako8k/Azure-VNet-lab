> **Disclaimer:** All Azure resources created in this lab were deleted after completion. No services were left running. This lab was built purely for learning and documentation purposes.

---
<img src="https://imgur.com/Eco5Dh6.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/jadNhKb.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/v0l5Pa1.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/AyrhCLE.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/DVDtrmf.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/ETiaz0m.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/jKOlO5d.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/My8EnP8.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/jKOlO5d.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/My8EnP8.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/DekbJKw.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/nF1SAoq.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/Ls1Hcop.png" height="80%" width="80%" alt=""/>




# Azure Virtual Network Lab — Hub-and-Spoke Topology

## Description

This lab focuses on how I built a hub-and-spoke network topology in Azure — covering VNet creation, peering across regions, Network Security Groups, and secure VM access via Azure Bastion.

| Detail | Value |
| --- | --- |
| **Resource Group** | `Az-Ropu1-lab` |
| **Hub Region** | Australia East |
| **Spoke Region** | Australia Southeast |
| **VNets** | `Neti-Hub` / `Neti-Spoke` |
| **Virtual Machines** | `Vmihini1` (Hub) / `Vmihini2` (Spoke) |
| **NSGs** | `Pirihi-mana-Hub` / `Pirihi-mana-Spoke` |
| **Bastion** | `Bastion-point` |

---

## What I Covered in This Lab

- Created two VNets in different regions and configured VNet peering in both directions
- Deployed VMs into subnets and applied NSGs with inbound/outbound rules
- Set up Azure Bastion for secure RDP/SSH access without public IPs
- Verified peering connectivity between `Neti-Hub` and `Neti-Spoke`
- Documented the topology with a network diagram

---

## Lab Structure

```
azure-vnet-lab/
├── README.md
├── commands-reference.md
└── screenshots/
    ├── part1-resource-group/
    ├── part2-vnets-and-subnets/
    ├── part3-vnet-peering/
    ├── part4-vms/
    ├── part5-nsgs/
    └── part6-bastion/
```

---

## Part 1 — Resource Group

I created a resource group named `Az-Ropu1-lab` in **Australia East** to house all resources for this lab.

**Steps:**

1. Navigate to **Resource groups** → **+ Create**
2. Name: `Az-Ropu1-lab` | Region: `Australia East`
3. Click **Review + create** → **Create**

<!-- screenshot -->

---

## Part 2 — VNets and Subnets

<!-- describe what you did and why here -->

| VNet | Region | Address Space | Subnets |
| --- | --- | --- | --- |
| `Neti-Hub` | Australia East | `10.1.0.0/16` | `SubnetHub` — `10.1.1.0/24` / `AzureBastionSubnet` — `10.1.2.0/26` |
| `Neti-Spoke` | Australia Southeast | `10.2.0.0/16` | `SubnetSpoke` — `10.2.1.0/24` |

> `AzureBastionSubnet` must be named exactly — Azure enforces this. The subnet range must be a minimum of `/26`.

**Steps:**

1. Navigate to **Virtual networks** → **+ Create**
2. Create `Neti-Hub` in `Australia East`
   - Address space: `10.1.0.0/16`
   - Add `SubnetHub` — `10.1.1.0/24`
   - Add `AzureBastionSubnet` — `10.1.2.0/26`
3. Create `Neti-Spoke` in `Australia Southeast`
   - Address space: `10.2.0.0/16`
   - Add `SubnetSpoke` — `10.2.1.0/24`

<!-- screenshot -->

---

## Part 3 — VNet Peering

<!-- describe what you did and why here -->

| Peering Name | Direction |
| --- | --- |
| `Neti-Hub-to-Spoke` | Hub → Spoke |
| `Neti-Spoke-to-Hub` | Spoke → Hub |

> Peering must be configured in both directions. Azure lets you create both in a single operation from one side.

**Steps:**

1. Open `Neti-Hub` → **Peerings** → **+ Add**
2. This side link name: `Neti-Hub-to-Spoke`
3. Remote virtual network: `Neti-Spoke`
4. Remote link name: `Neti-Spoke-to-Hub`
5. Click **Add**
6. Verify both show as **Connected** in each VNet's Peerings blade

<!-- screenshot -->

---

## Part 4 — Virtual Machines

<!-- describe what you did and why here -->

| VM | VNet | Subnet | OS | Public IP |
| --- | --- | --- | --- | --- |
| `Vmihini1` | `Neti-Hub` | `SubnetHub` | Windows Server 2022 | None |
| `Vmihini2` | `Neti-Spoke` | `SubnetSpoke` | Windows Server 2022 | None |

> No public IPs were assigned — access is handled entirely through Bastion.

**Steps:**

1. Navigate to **Virtual machines** → **+ Create**
2. Deploy `Vmihini1` into `Az-Ropu1-lab`, region `Australia East`
   - VNet: `Neti-Hub` / Subnet: `SubnetHub`
   - Public IP: None | NIC NSG: None
3. Deploy `Vmihini2` into `Az-Ropu1-lab`, region `Australia Southeast`
   - VNet: `Neti-Spoke` / Subnet: `SubnetSpoke`
   - Public IP: None | NIC NSG: None

<!-- screenshot -->

---

## Part 5 — Network Security Groups (NSGs)

<!-- describe what you did and why here -->

| NSG | Attached To |
| --- | --- |
| `Pirihi-mana-Hub` | `SubnetHub` — `Neti-Hub` |
| `Pirihi-mana-Spoke` | `SubnetSpoke` — `Neti-Spoke` |

**Inbound Rules — Pirihi-mana-Hub:**

| Priority | Name | Port | Protocol | Source | Action |
| --- | --- | --- | --- | --- | --- |
| 100 | Allow-RDP | 3389 | TCP | Any | Allow |
| 110 | Allow-ICMP | * | ICMP | `10.2.0.0/16` | Allow |
| 4096 | Deny-All-Inbound | * | Any | Any | Deny |

**Inbound Rules — Pirihi-mana-Spoke:**

| Priority | Name | Port | Protocol | Source | Action |
| --- | --- | --- | --- | --- | --- |
| 100 | Allow-RDP | 3389 | TCP | Any | Allow |
| 110 | Allow-ICMP | * | ICMP | `10.1.0.0/16` | Allow |
| 4096 | Deny-All-Inbound | * | Any | Any | Deny |

> The ICMP rule source is swapped between NSGs — Hub allows ICMP from the Spoke range and vice versa. This is what permits the ping test across peered VNets.

**Steps:**

1. Navigate to **Network security groups** → **+ Create**
2. Create `Pirihi-mana-Hub` in `Australia East`, associate to `SubnetHub`
3. Create `Pirihi-mana-Spoke` in `Australia Southeast`, associate to `SubnetSpoke`
4. Add inbound rules to each as per the tables above

<!-- screenshot -->

---

## Part 6 — Azure Bastion

<!-- describe what you did and why here -->

| Detail | Value |
| --- | --- |
| **Name** | `Bastion-point` |
| **VNet** | `Neti-Hub` |
| **Subnet** | `AzureBastionSubnet` |
| **Tier** | Basic |
| **Public IP** | `Bastion-point-pip` |

> Bastion provides browser-based RDP/SSH through the Azure Portal — no client software or public IP on the VM is required.

**Steps:**

1. Navigate to **Bastions** → **+ Create**
2. Assign to `Az-Ropu1-lab`, region `Australia East`
3. Virtual network: `Neti-Hub` — subnet auto-detects `AzureBastionSubnet`
4. Create new public IP named `Bastion-point-pip`
5. Tier: **Basic**
6. Click **Review + create** → **Create**

> **Note:** VM access via Bastion was not completed. The VMs were flagged during deployment due to security compliance restrictions on the subscription. Connectivity testing across the peered VNets (Part 7) was skipped as a result.

<!-- screenshot -->

---

## Part 7 — Connectivity Test

> Skipped. VM access was restricted due to security compliance policies on the subscription. Bastion was successfully deployed and configured. The peering between `Neti-Hub` and `Neti-Spoke` was verified as **Connected** on both sides.

---

## Cleanup

Resources were deleted in the following order to avoid dependency conflicts:

1. **Bastion** — `Bastion-point`
2. **Virtual Machines** — `Vmihini1` and `Vmihini2` (OS disks and NICs deleted with VMs)
3. **NSGs** — `Pirihi-mana-Hub` and `Pirihi-mana-Spoke`
4. **VNet Peerings** — deleted from `Neti-Hub` (return peering auto-removed)
5. **VNets** — `Neti-Hub` and `Neti-Spoke`
6. **Public IP** — `Bastion-point-pip`
7. **Resource Group** — `Az-Ropu1-lab` (catches any remaining orphaned resources)

---

## Network Diagram

<!-- insert diagram here -->

---

## Key Takeaways

<!-- write this in your own words after completing the lab -->

---

## Lessons Learned

<!-- write this in your own words after completing the lab -->

---

*Lab completed by [@Nako8k](https://github.com/Nako8k)*
