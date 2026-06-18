# Azure Virtual Network Lab — Hub-and-Spoke Topology
> **Disclaimer:** All Azure resources created in this lab were deleted after completion. No services were left running. This lab was built purely for learning and documentation purposes.

---

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
<br /> 
<img src="https://imgur.com/Eco5Dh6.png" height="80%" width="80%" alt=""/>

---

## Part 2 — VNets and Subnets

I created two Vnets which I named `Neti-Hub` in **Australia East** and `Neti-Spoke` in **Australia Southeast**, 
within the Virtual Networks I created 3 subnets `SubnetHub`, and `AzureBastionSubnet` which were setup in **Neti-Hub**, and 
`SubnetSpoke` which is setup in **Neti-Spoke**

| VNet | Region | Address Space | Subnets |
| --- | --- | --- | --- |
| `Neti-Hub` | Australia East | `10.1.0.0/16` | `SubnetHub` — `10.1.1.0/24` / `AzureBastionSubnet` — `10.1.2.0/26` |
| `Neti-Spoke` | Australia Southeast | `10.2.0.0/16` | `SubnetSpoke` — `10.2.1.0/24` |

> `AzureBastionSubnet` must be named exactly — Azure enforces this. The subnet range must be a minimum of `/26` aswell.

**Steps:**

1. Navigate to **Virtual networks** → **+ Create**
2. Create `Neti-Hub` in `Australia East`
   - Address space: `10.1.0.0/16`
   - Add `SubnetHub` — `10.1.1.0/24`
   - Add `AzureBastionSubnet` — `10.1.2.0/26`
3. Create `Neti-Spoke` in `Australia Southeast`
   - Address space: `10.2.0.0/16`
   - Add `SubnetSpoke` — `10.2.1.0/24`
<br /> 
<img src="https://imgur.com/jadNhKb.png" height="80%" width="80%" alt=""/>

---

## Part 3 — VNet Peering

Once both VNets were configured, I set up VNet peering, which allowed the two VNets to connect and communicate with each other.

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
<br /> 
<img src="https://imgur.com/v0l5Pa1.png" height="80%" width="80%" alt=""/>
<br /> 
<img src="https://imgur.com/AyrhCLE.png" height="80%" width="80%" alt=""/>

---

## Part 4 — Virtual Machines

I created two VMs `Vmihini1` that would sit in **Neti-Hub** and `Vmihini2` which I put into **Neti-Spoke**

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
<br /> 
<img src="https://imgur.com/DVDtrmf.png" height="80%" width="80%" alt=""/>
<br /> 
<img src="https://imgur.com/ETiaz0m.png" height="80%" width="80%" alt=""/>

---

## Part 5 — Network Security Groups (NSGs)

Once the VMs were deployed, I configured Network Security Groups (NSGs) and inbound security rules to control traffic entering resources in both VNets.


| NSG | Attached To |
| --- | --- |
| `Pirihi-mana-Hub` | `SubnetHub` — `Neti-Hub` |
| `Pirihi-mana-Spoke` | `SubnetSpoke` — `Neti-Spoke` |


> The ICMP rules were configured to allow traffic from the opposite VNet range, enabling ping tests across the peered VNets.
However, I mistakenly assigned ports (443 and 21) to the ICMP rules. Since ICMP does not use ports, the ping test failed until the rules were corrected hinting towards why I skipped the final section.


**Inbound Rules — Pirihi-mana-Hub:**

| Priority | Name | Port | Protocol | Source | Action |
| --- | --- | --- | --- | --- | --- |
| 100 | Allow-RDP | 3389 | TCP | Any | Allow |
| 110 | Allow-ICMP | 21 | ICMP | `10.2.0.0/16` | Allow |
| 4096 | Deny-All-Inbound | 25 | Any | Any | Deny |

**Inbound Rules — Pirihi-mana-Spoke:**

| Priority | Name | Port | Protocol | Source | Action |
| --- | --- | --- | --- | --- | --- |
| 100 | Allow-RDP | 3389 | TCP | Any | Allow |
| 110 | Allow-ICMP | 443 | ICMP | `10.1.0.0/16` | Allow |
| 4096 | Deny-All-Inbound | 443 | Any | Any | Deny |
<br /> 
<img src="https://imgur.com/jKOlO5d.png" height="80%" width="80%" alt=""/>
<br /> 
<img src="https://imgur.com/My8EnP8.png" height="80%" width="80%" alt=""/>

---

## Part 6 — Azure Bastion

I created an Azure Bastion to securely connect to the VMs and verify connectivity between the peered VNets.

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

> **Note:** VM access via Bastion was successful. Although both VMs connected correctly, the ping test failed due to the NSG misconfiguration described in Part 5.

<img src="https://imgur.com/DekbJKw.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/nF1SAoq.png" height="80%" width="80%" alt=""/>

<img src="https://imgur.com/Ls1Hcop.png" height="80%" width="80%" alt=""/>

---

## Part 7 — Connectivity Test

> VM access via Bastion was successful, and the peering between `Neti-Hub` and `Neti-Spoke` was verified as **Connected** on both sides. However, connectivity testing initially failed due to an NSG misconfiguration, where ICMP rules were incorrectly assigned port numbers. This issue highlighted the importance of carefully validating security rules and serves as a valuable lesson for future Azure networking projects.

---

## Key Takeaways



---

## Lessons Learned



---

*Lab completed by [@Nako8k](https://github.com/Nako8k)*
