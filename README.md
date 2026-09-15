# Cybersecurity Homelab

This repository documents my personal homelab as I build hands-on experience with cybersecurity, networking, virtualization, Linux administration, and self-hosted infrastructure.

The environment is built around Proxmox VE and is being expanded through individual projects covering infrastructure, self-hosted services, secure remote access, and eventually dedicated cybersecurity testing and monitoring environments.

## Current Environment

### Hardware

- Dell Vostro 3420
- Intel Core i7-1255U
- 16 GB RAM
- 512 GB NVMe SSD

### Infrastructure

- Proxmox VE 9
- Debian and Ubuntu Linux
- LXC containers
- Docker and Docker Compose
- Home LAN: `192.168.1.0/24`

## Current Homelab

```text
                         Internet
                            |
                       Home Router
                            |
                  192.168.1.0/24 LAN
                            |
                     Proxmox VE Host
                            |
          +-----------------+-----------------+
          |                 |                 |
     Ubuntu Server       Pi-hole         Vaultwarden
        VM 100            CT 101            CT 102
                           |                  |
                           |             Docker Compose
                           |                  |
                           |          +-------+-------+
                           |          |               |
                       Local DNS  Vaultwarden       Caddy
                           |                      HTTPS Proxy
                           |
                    Network-Wide DNS

                       Tailscale
                         CT 103
                           |
                    Secure Remote Access
                           |
                    Tailscale Clients
```
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
- [ ] Automate backups for critical services
- [ ] Create an isolated cybersecurity testing network
- [ ] Deploy Kali Linux and vulnerable target systems
- [ ] Build a Windows Active Directory environment
- [ ] Deploy centralized security monitoring with Wazuh
- [ ] Develop automated detection and response workflows
- [ ] Build dedicated NAS, media, and backup infrastructure
