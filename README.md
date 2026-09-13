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
  
## Next Steps

- Begin experimenting with Linux administration on the Ubuntu Server VM.
- Deploy the first dedicated homelab service.
- Build an isolated cybersecurity testing network.
- Add attacker and target virtual machines for security testing.
