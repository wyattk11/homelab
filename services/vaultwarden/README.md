## Self-Hosted Vaultwarden Password Manager

Deployed Vaultwarden as a self-hosted password management service to gain hands-on experience with Docker, HTTPS/TLS, reverse proxies, local DNS, SSH administration, and backup procedures.

### Deployment

- Created a Debian 13 LXC container in Proxmox
- Allocated 1 vCPU, 512 MB RAM, and 8 GB storage
- Configured a DHCP reservation for the Vaultwarden server at `192.168.1.111`
- Installed and enabled Docker inside the LXC container
- Verified container functionality using the Docker `hello-world` image
- Deployed Vaultwarden using Docker Compose
- Configured persistent storage to preserve Vaultwarden data across container recreation

### HTTPS and Local DNS

- Deployed Caddy as a reverse proxy in front of Vaultwarden
- Configured Caddy to provide HTTPS using an internal certificate authority
- Removed direct LAN exposure of the Vaultwarden container and routed access through Caddy
- Created a Pi-hole local DNS record mapping `vault.home.arpa` to the Vaultwarden server
- Installed the Caddy root certificate on a Windows client to establish trust for the internal certificate authority
- Verified secure access to Vaultwarden at `https://vault.home.arpa`

### SSH Administration and Hardening

- Installed and enabled OpenSSH on the Vaultwarden server
- Created a dedicated administrative Linux user with sudo privileges
- Configured Ed25519 public-key authentication using an existing client SSH key
- Disabled new Vaultwarden account registrations after creating the primary account

### Backup and Recovery Preparation

- Identified `/opt/vaultwarden/data/` as the persistent application data directory
- Created a consistent backup by stopping Vaultwarden before archiving its persistent data
- Transferred the backup to a separate Windows system using SCP
- Verified Vaultwarden successfully restarted after the backup procedure
- Kept sensitive Vaultwarden backup data separate from the public GitHub repository

### Client Configuration and Password Migration

- Configured a Windows client to trust the Caddy internal certificate authority
- Configured an iPhone to use Pi-hole for local DNS resolution
- Installed and trusted the Caddy internal root certificate on iOS
- Verified secure Vaultwarden access from both Windows and iOS
- Connected the Bitwarden mobile application to the self-hosted Vaultwarden server
- Enabled Face ID authentication and password AutoFill on iOS
- Migrated existing credentials from Brave into Vaultwarden
- Migrated saved iOS credentials from Apple Passwords into Vaultwarden
- Retained the original password stores temporarily to validate the migration before removing duplicate credentials

### Secure Remote Access

Remote access to Vaultwarden was configured using a dedicated Tailscale subnet router rather than exposing the service directly to the public Internet.

- Connected an iPhone to the private Tailscale network
- Configured a Tailscale subnet router to provide access to the `192.168.1.0/24` home network
- Configured split DNS for the `home.arpa` domain using Pi-hole
- Verified that `vault.home.arpa` resolves correctly while connected remotely
- Tested Vaultwarden access over cellular with Wi-Fi disabled
- Verified that the existing HTTPS configuration remains trusted remotely
- Avoided router port forwarding or direct public exposure of Vaultwarden

Vaultwarden can now be accessed using the same `https://vault.home.arpa` address both locally and remotely through Tailscale.

### Result

Vaultwarden is successfully running as a Docker container inside a lightweight Proxmox LXC environment. The service uses Caddy as an HTTPS reverse proxy, Pi-hole for local DNS resolution, and persistent Docker storage for application data.

Secure access has been validated from both Windows and iOS clients. Existing credentials were migrated from Brave and Apple Passwords, and the Bitwarden mobile application is configured with Face ID and AutoFill.

Remote access is provided through an authenticated Tailscale connection without exposing Vaultwarden directly to the public Internet. The deployment also includes SSH key-based administration, restricted account registration, and an off-host backup of Vaultwarden data.
