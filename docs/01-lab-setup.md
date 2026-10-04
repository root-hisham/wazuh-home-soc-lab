# 1. Lab Setup

## Hypervisor and VMs

All VMs run in **Oracle VirtualBox** on a Windows host (16 GB RAM).

| VM | OS | RAM | CPUs | Role |
|---|---|---|---|---|
| wazuh-server | Ubuntu Server 22.04.5 LTS | 4 GB | 2 | SIEM (manager, indexer, dashboard) |
| ubuntu1 | Ubuntu 25.04 | 2 GB | 2 | Monitored endpoint |
| kali | Kali Linux | 2 GB | 2 | Attacker |

VM disks are stored on a secondary drive to avoid filling the system drive, using dynamically allocated disks so they only grow as needed.

## Isolated network

Instead of VirtualBox's default NAT (which isolates each VM from the others), a **NAT Network** named `soc-lab` was created:

- **File → Tools → Network Manager → NAT Networks → Create**
- Name: `soc-lab`
- IPv4 prefix: `10.0.3.0/24`
- DHCP: enabled

All three VMs have their network adapter set to **Attached to: NAT Network → soc-lab**, so they can reach each other and the internet, but stay isolated from the host's own LAN.

Final addressing:

| Host | IP |
|---|---|
| wazuh-server | 10.0.3.8 |
| ubuntu1 | 10.0.3.7 |
| kali | 10.0.3.4 |

## Accessing the lab from the host

Two VirtualBox port-forwarding rules on `soc-lab` make the lab reachable from the Windows host:

| Name | Protocol | Host | Guest | Purpose |
|---|---|---|---|---|
| wazuh-dashboard | TCP | 127.0.0.1:8443 | 10.0.3.8:443 | Open the Wazuh dashboard in a normal browser |
| ssh-wazuh (optional) | TCP | 127.0.0.1:2223 | 10.0.3.8:22 | SSH in instead of typing long commands into the VM console |

## Snapshots

A snapshot was taken after each major milestone (clean OS install, before installing Wazuh, after the agent connected) so any failed step could be rolled back without rebuilding the VM from scratch.
