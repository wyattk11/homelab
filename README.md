# Homelab

This repository documents the ongoing development of my personal homelab. The environment will primarily be used for cybersecurity labs, networking, virtualization, and testing self-hosted services.

## Hardware

**Dell Vostro 3420**
- Intel Core i7-1255U
- 16 GB RAM
- 512 GB NVMe SSD
- Proxmox VE 9.2

## Progress

### 09/13/2026 - Initial Proxmox Setup

- Repurposed an unused Dell Vostro laptop as a dedicated homelab server.
- Removed Windows and installed Proxmox VE 9.2 as the bare-metal hypervisor.
- Configured the Proxmox host with a static management IP.
- Configured the laptop to remain operational with the lid closed for headless use.
- Verified the Proxmox host remained running after closing the lid.

### 09/13/2026 - Proxmox Configuration & First VM

- Configured Proxmox VE for headless operation and verified remote management from my desktop.
- Configured the Proxmox no-subscription repository and updated the host.
- Created my first virtual machine using Ubuntu Server.
- Allocated 2 vCPUs, 2 GB RAM, and a 32 GB virtual disk to the VM.
- Configured bridged networking through `vmbr0`, allowing the VM to communicate with my home network.
- Enabled SSH and successfully connected to the Ubuntu VM remotely from my desktop.
- Installed and configured the QEMU Guest Agent for improved communication between Proxmox and the VM.
- Created a `clean-install` snapshot to provide a known-good rollback point before future testing.
  
## Project 1: Pi-hole Network DNS Filtering

Deployed Pi-hole as the first self-hosted service in my homelab to provide network-level DNS filtering and gain hands-on experience with DNS, DHCP, containers, and network configuration.

### Deployment

- Created a Debian 13 LXC container in Proxmox
- Allocated 1 vCPU, 512 MB RAM, and 8 GB storage
- Connected the container directly to the LAN through the Proxmox `vmbr0` bridge
- Configured a DHCP reservation for the Pi-hole server at `192.168.1.220`
- Installed and configured Pi-hole
- Configured Cloudflare as the upstream DNS provider with DNSSEC
- Enabled query logging for DNS traffic visibility
- Added the StevenBlack Unified Hosts blocklist
- Generated a gravity database containing approximately 80,000 blocked domains

### Testing

Configured a Windows client to use `192.168.1.220` as its DNS server and verified that DNS requests were being processed by Pi-hole.

Normal DNS resolution was successfully verified:

`google.com` → Successfully resolved through Pi-hole

DNS filtering was then tested using an advertising/tracking domain:

`doubleclick.net` → Returned `0.0.0.0` / `::`, confirming the request was blocked by Pi-hole

### Troubleshooting and SSH Hardening

- Investigated unexpected IPv6 DNS behavior after the Windows client initially bypassed Pi-hole
- Traced DNS configuration across the Windows client, Proxmox host, LXC container, and router
- Confirmed the home network currently does not have configured IPv6 Internet connectivity and removed unnecessary client-side IPv6 DNS configuration
- Created a dedicated administrative Linux user with sudo privileges
- Installed and enabled OpenSSH for remote administration
- Configured Ed25519 public-key authentication
- Disabled SSH password authentication after validating key-based access
- Disabled direct root SSH login
- Validated SSH configuration with `sshd -t` before reloading the service

### Result

Pi-hole is successfully running as a lightweight LXC service on the Proxmox homelab. DNS resolution and filtering were validated from a Windows client before any network-wide DNS changes were made. Secure remote administration was also configured using SSH key authentication and a dedicated sudo-enabled user.

A working-state Proxmox snapshot was created after successful deployment and hardening.

## Next Steps

- Deploy Pi-hole DNS filtering across the home network
- Continue building lightweight self-hosted services using LXC containers
- Create an isolated virtual network for cybersecurity testing
- Deploy a Kali Linux attack VM and vulnerable target systems
- Build a small Windows Active Directory lab
- Add centralized security monitoring with a SIEM such as Wazuh
- Explore automated detection and response workflows
- Plan a future dedicated server for NAS, media hosting, backups, and production self-hosted services
