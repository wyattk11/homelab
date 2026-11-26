# Isolated Cybersecurity Range

## Overview

Built an isolated virtual cybersecurity lab within my Proxmox homelab to provide a controlled environment for future security testing, vulnerable systems, Active Directory labs, and security monitoring projects.

The range is separated from the primary home network using a dedicated Proxmox virtual bridge and an OPNsense firewall.

## Architecture

Home Network (`192.168.1.0/24`)  
↓  
`vmbr0`  
↓  
OPNsense Firewall  
↓  
`vmbr1`  
↓  
Cyber Range (`10.10.10.0/24`)  
↓  
Kali Linux  
↓  
Vulnerable Target Systems

## Network Isolation

Created a dedicated Proxmox Linux bridge named `vmbr1` for cybersecurity lab traffic.

- Configured `vmbr1` as a Layer 2 bridge with no IP address assigned to the Proxmox host
- Kept the cyber range separate from the existing `vmbr0` home network
- Assigned the cyber range the `10.10.10.0/24` subnet
- Prevented lab systems from directly accessing the primary `192.168.1.0/24` home network

Keeping the Proxmox host itself off the cyber-range subnet reduces unnecessary exposure of the hypervisor to systems used for security testing.

## OPNsense Firewall

Deployed OPNsense as the gateway between the isolated cyber range and the home network.

### Virtual Machine

- VM ID: 201
- Name: `lab-firewall`
- 2 vCPU
- 2 GB RAM
- 16 GB storage
- OPNsense 26.7

### Interfaces

WAN:

- Connected to `vmbr0`
- Receives an address from the home network through DHCP

LAN:

- Connected to `vmbr1`
- Address: `10.10.10.1/24`

OPNsense provides DHCP services to the cyber range using:

`10.10.10.100 - 10.10.10.199`

The firewall performs routing and NAT so lab systems can access the Internet when needed without being directly connected to the home network.

## Firewall Rules

Replaced the original two-rule setup with a default-deny policy on the OPNsense LAN interface. Rules are evaluated top to bottom:

1. Block `LAN net` to `192.168.1.0/24`, so nothing in the range can reach the home network
2. Allow Kali (`10.10.10.102`) to any destination, for updates and tool downloads
3. Allow `LAN net` to the firewall itself, for DNS
4. Allow target-web to any destination (temporary, disabled by default)
5. Block `LAN net` to any destination, with logging enabled

IPv6 is currently not configured for the cyber range.

- Backed up the OPNsense configuration before changing any rules
- Kept the temporary target-web rule disabled. It is only enabled to patch or rebuild that VM, then disabled again
- Logged the final block rule so denied traffic from the targets shows up in the firewall logs
- Result: Kali keeps Internet access, but the vulnerable targets have no route to the Internet or the home LAN

## Kali Linux

Deployed Kali Linux as the primary attacker and security-testing workstation.

### Virtual Machine

- VM ID: 200
- Name: `kali`
- 2 vCPU
- 2 GB RAM
- 32 GB storage
- Connected only to `vmbr1`

Kali receives its network configuration from the OPNsense DHCP server.

Connectivity testing confirmed:

- Internet connectivity through OPNsense
- Isolation from the primary home network

For example, Kali successfully reached an external Internet address while traffic to the home router at `192.168.1.1` was blocked by the OPNsense firewall.

## Target Systems

Deployed two intentionally vulnerable targets on `vmbr1` for scanning and assessment practice.

### target-web (VM 202)

- Ubuntu Server 22.04 cloud image, 1 vCPU, 1 GB RAM, 10 GB storage
- Static address `10.10.10.50`
- Built with a cloud-init user-data snippet, which mirrors how the original AWS EC2 version of this target was bootstrapped
- Verified the cloud image against Ubuntu's published SHA256 checksums before importing it
- Deliberately weak configuration:
  - Apache with the default configuration and no security headers
  - vsftpd with anonymous login enabled
  - SSH password authentication enabled, with a weak test account
  - Automatic security updates disabled
- Installed the QEMU guest agent so the VM can be managed from the Proxmox host without network access

### target-msf (VM 203)

- Metasploitable 2 (Rapid7), 1 vCPU, 512 MB RAM, 8 GB storage
- Receives its address from OPNsense DHCP
- Used an emulated e1000 NIC and an IDE disk, because the old kernel does not support VirtIO
- Verified the download against SourceForge's published MD5 and SHA-1 hashes before importing it

### Snapshots

- Took a `clean-build` snapshot of both targets before any testing
- Targets can be rolled back to a known-clean state after each exercise

## Current Architecture

```
                    Internet
                       |
                  Home Router
                  192.168.1.1
                       |
                     vmbr0
                       |
               +---------------+
               |   OPNsense    |
               | VM 201        |
               |               |
               | WAN     LAN   |
               +---------+-----+
                         |
                       vmbr1
                         |
                  10.10.10.0/24
                         |
        +----------------+----------------+
        |                |                |
   +---------+     +------------+   +------------+
   |  Kali   |     | target-web |   | target-msf |
   | VM 200  |     | VM 202     |   | VM 203     |
   +---------+     +------------+   +------------+
```

## Verification

Tested isolation from each system after the new rules were applied.

From Kali:
- Internet access worked
- Traffic to the home LAN (`192.168.1.0/24`) was blocked

From target-web, run through the QEMU guest agent:
- Outbound HTTPS to the Internet was blocked
- Traffic to a home LAN service was blocked
- DNS resolution through OPNsense still worked

From target-msf:
- Checked the OPNsense state table and found no connections leaving the range
- Confirmed the default-deny rule was logging its blocked traffic

## Troubleshooting: Filtered Port Results

During the first Nmap scan from Kali, every closed port on both targets showed as `filtered` instead of `closed`. Metasploitable has no host firewall, so something in the path was dropping the TCP resets that mark a port as closed.

- Captured traffic on the Proxmox host at the target's tap interface. The target was sending RST replies to Kali's probes
- Captured on Kali's tap interface. The same RST replies never arrived
- Found the Proxmox firewall checkbox enabled on the NICs of all lab VMs. Traffic between two VMs on the same host passed through Proxmox connection tracking twice, and the second pass treated the RST as an invalid packet and dropped it
- Confirmed this with the host's conntrack statistics, which showed a large number of invalid packets
- No Proxmox VM firewall rules were defined for these VMs, and OPNsense already enforces isolation for the range. Disabled the per-NIC Proxmox firewall option on the `vmbr1` interfaces of the four lab VMs
- Re-ran the scan: closed ports now correctly showed as `closed`
- Re-ran the isolation tests to confirm OPNsense was still blocking the targets

Takeaway: a `filtered` result does not always mean a firewall in front of the target. It can come from your own infrastructure, and packet captures at both ends show where the traffic is actually lost.

## Management Access

SSH access to the homelab was updated after migrating to a new desktop system.

- Restored SSH key-based authentication
- Added SSH aliases for Proxmox, Pi-hole, CraftLab, Vaultwarden, and Tailscale
- Verified passwordless key authentication to configured services

## Future Use

The isolated range will serve as the foundation for future cybersecurity projects, including:

- Vulnerable Windows targets
- Active Directory security labs
- Network traffic analysis
- Penetration testing exercises
- Wazuh security monitoring
- Detection and response automation

Individual cybersecurity exercises and implementations performed inside this environment are documented separately under the repository's `projects` directory:

- [Cyber Range Reconnaissance and Vulnerability Assessment](../../projects/cyber-range-recon/README.md)
