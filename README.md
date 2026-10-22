
# Cybersecurity Homelab

This repository documents my personal homelab as I build hands-on experience with cybersecurity, networking, virtualization, Linux administration, and self-hosted infrastructure.

The environment is built around two independent Proxmox VE hosts and is being expanded through individual projects covering infrastructure, self-hosted services, secure remote access, and dedicated cybersecurity testing and monitoring environments.

## Current Environment

### Hardware

**Primary Infrastructure Host — Dell Vostro 3420 (`pve`)**
- Intel Core i7-1255U (10 cores / 12 threads)
- 16 GB RAM
- 512 GB NVMe SSD
- Proxmox VE 9
- Hosts infrastructure services and isolated cybersecurity lab environments

**Secondary Compute Host — Custom Ryzen Server (`pve2`)**
- AMD Ryzen 9 5900X (12 cores / 24 threads)
- 32 GB DDR4 RAM
- ASUS ROG STRIX B550-A motherboard
- Samsung 860 EVO 1 TB SSD
- Proxmox VE 9.2
- Headless operation without a dedicated GPU
- Intended for resource-intensive virtual machines and future services

**Networking**
- Unmanaged Gigabit Ethernet switch
- Home LAN: `192.168.1.0/24`
- Dell management IP: `192.168.1.200`
- Ryzen management IP: `192.168.1.201`

Both Proxmox hosts operate independently on the same network rather than as a cluster.

### Infrastructure

- Proxmox VE 9
- Debian and Ubuntu Linux
- LXC containers and virtual machines
- Docker and Docker Compose
- OPNsense firewall
- Isolated virtual networking
- SSH key-based administration
- Proxmox host firewall rules
- Tailscale secure remote access

## Current Homelab

    Internet
       |
    Home Router
       |
    192.168.1.0/24 LAN
       |
    Gigabit Ethernet Switch
       |
       +-------------------------------+
       |                               |
    Dell Proxmox                    Ryzen Proxmox
    pve (.200)                      pve2 (.201)
       |                               |
       +-- Ubuntu Server (VM 100)      +-- Future Minecraft Server
       |
       +-- Pi-hole (CT 101)
       |      |
       |   Local DNS Filtering
       |
       +-- Vaultwarden (CT 102)
       |      |
       |   Docker + Caddy HTTPS
       |
       +-- Tailscale (CT 103)
       |      |
       |   Secure Remote Access
       |
       +-- Minecraft (VM 104)
       |      |
       |   Existing Better Fantasy Server
       |
       +-- Isolated Cybersecurity Range
              |
              +-- OPNsense (VM 201)
              |      |
              |   Firewall / Network Isolation
              |
              +-- Kali Linux (VM 200)
                     |
                  Security Testing

## Security Configuration

Implemented additional security controls across both Proxmox hosts to protect management access and support secure administration.

- Configured SSH key-based authentication on the Ryzen host.
- Disabled password-based SSH authentication on the Ryzen host.
- Restricted root SSH access to public-key authentication.
- Enabled the built-in Proxmox firewall on both hosts.
- Restricted SSH (TCP 22) and Proxmox Web UI (TCP 8006) management access to the home LAN.
- Configured a default DROP policy for incoming host traffic.
- Reviewed existing guest firewall configurations before enabling the Dell host firewall.
- Verified Pi-hole DNS functionality following firewall activation.
- Maintained separation between the home network and isolated cybersecurity lab.

## Homelab Roadmap

- [x] Deploy Proxmox virtualization environment
- [x] Deploy Ubuntu Server VM
- [x] Deploy Pi-hole DNS filtering
- [x] Expand Pi-hole to network-wide DNS
- [x] Deploy Vaultwarden password manager
- [x] Configure HTTPS and local DNS
- [x] Configure Windows and iOS Vaultwarden clients
- [x] Migrate existing credentials into Vaultwarden
- [x] Configure secure remote access with Tailscale
- [x] Create an isolated cybersecurity testing network
- [x] Deploy Kali Linux and OPNsense firewall
- [x] Configure network isolation between the cybersecurity range and home LAN
- [x] Deploy a modded Minecraft server
- [x] Deploy second Proxmox virtualization host
- [x] Configure headless operation on Ryzen server
- [x] Implement Proxmox host firewall rules
- [x] Harden SSH authentication on Ryzen server
- [ ] Deploy All the Mods 10 Minecraft server on Ryzen host
- [ ] Automate backups for critical services
- [ ] Deploy vulnerable target systems for cybersecurity testing
- [ ] Build a Windows Active Directory environment
- [ ] Deploy centralized security monitoring with Wazuh
- [ ] Develop automated detection and response workflows
- [ ] Build dedicated NAS, media, and backup infrastructure
