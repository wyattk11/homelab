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

## Next Steps

- Connect the Proxmox host directly to the home network.
- Access and configure the Proxmox web interface.
- Deploy the first virtual machine.
