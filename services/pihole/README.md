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

### Network-Wide Deployment

After validating Pi-hole on individual clients, the router DHCP configuration was updated to distribute `192.168.1.220` as the DNS server across the home network.

- Configured the router to provide Pi-hole as the DNS server through DHCP
- Returned previously configured clients to automatic DNS configuration
- Renewed client network connections and verified `192.168.1.220` was automatically assigned as the DNS server
- Expanded Pi-hole from individual client testing to network-wide DNS filtering
- Continued using Pi-hole for local DNS resolution of homelab services such as `vault.home.arpa`

This allows devices joining the home network to automatically use Pi-hole without requiring manual DNS configuration on each client.

### Network-Wide Troubleshooting

After deploying Pi-hole as the network-wide DNS server, an iPhone connected to the network experienced Wi-Fi connectivity but was unable to access the Internet, while other devices continued working normally.

I verified that the device received a valid DHCP configuration, including an IP address within the `192.168.1.0/24` network, the `192.168.1.1` default gateway, and `192.168.1.220` as its DNS server.

Further investigation showed that DNS requests on the iPhone were being routed through iCloud Private Relay. Disabling **Limit IP Address Tracking** for the home Wi-Fi network disabled Private Relay for that network and immediately restored connectivity.

This provided practical experience troubleshooting a client-specific network issue by verifying DHCP, gateway, DNS, local connectivity, and client configuration individually rather than assuming the DNS server itself was unavailable.

### Result

Pi-hole is successfully running as a lightweight LXC service on the Proxmox homelab and now provides network-wide DNS filtering through the router's DHCP configuration.

In addition to filtering external DNS requests, Pi-hole provides local DNS resolution for other homelab services, including the Vaultwarden deployment. This allows internal services to use consistent hostnames instead of requiring direct IP addresses.

The deployment includes DNS filtering, local DNS, query logging, SSH key-based administration, and integration with other homelab infrastructure.
