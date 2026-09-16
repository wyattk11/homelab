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
Future Lab Systems

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

Configured the cyber-range firewall so lab systems can reach the Internet while preventing access to the primary home network.

Rules are evaluated in order:

1. Block traffic from `LAN network` to `192.168.1.0/24`
2. Allow IPv4 traffic from `LAN network` to other destinations

IPv6 is currently not configured for the cyber range.

This allows systems inside the range to download updates and tools while preventing them from communicating with devices on the primary home LAN.

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

```## Current Architecture

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
                    +---------+
                    |  Kali   |
                    | VM 200  |
                    +---------+
```
## Verification

The completed environment was tested from Kali Linux.

External connectivity:

`ping 8.8.8.8`

Successful responses confirmed that OPNsense routing and NAT provide Internet connectivity to the isolated range.

Home network isolation:

`ping 192.168.1.1`

No responses were received, confirming that the firewall rule prevents the cyber range from accessing the primary home network.

## Future Use

The isolated range will serve as the foundation for future cybersecurity projects, including:

- Vulnerable Linux and Windows targets
- Active Directory security labs
- Network traffic analysis
- Vulnerability assessment
- Penetration testing exercises
- Wazuh security monitoring
- Detection and response automation

Individual cybersecurity exercises and implementations performed inside this environment will be documented separately under the repository's `projects` directory.
