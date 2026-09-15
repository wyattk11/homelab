## Pi-hole Network DNS Filtering

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
