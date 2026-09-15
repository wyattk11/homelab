# Tailscale Remote Access

## Overview

Deployed a dedicated Tailscale gateway to provide secure remote access to internal homelab services without exposing those services directly to the public Internet.

## Deployment

- Created an unprivileged Debian 13 LXC container in Proxmox
- Allocated 1 vCPU, 512 MB RAM, and 8 GB storage
- Enabled nesting
- Passed `/dev/net/tun` into the LXC container
- Installed and enabled Tailscale
- Joined the gateway to a private Tailscale network

## Subnet Routing

Configured the Tailscale container as a subnet router for:

`192.168.1.0/24`

- Enabled IPv4 forwarding on the Linux gateway
- Advertised the home LAN through Tailscale
- Approved the advertised subnet through the Tailscale administration console
- Connected an iPhone to the same Tailscale network
- Verified remote LAN connectivity over cellular by accessing Pi-hole at `192.168.1.220`

## Split DNS

Configured Tailscale split DNS so requests for:

`home.arpa`

are sent to the Pi-hole DNS server at:

`192.168.1.220`

This allows internal services such as:

`vault.home.arpa`

to use the same hostname both inside and outside the home network.

Public DNS traffic remains separate from the homelab-specific DNS configuration.

## Vaultwarden Remote Access

Verified Vaultwarden access from an iPhone using cellular connectivity with Wi-Fi disabled.

Remote traffic follows:

iPhone
→ Tailscale encrypted tunnel
→ Tailscale subnet router
→ Home LAN
→ Pi-hole local DNS
→ Caddy HTTPS reverse proxy
→ Vaultwarden

Vaultwarden remains inaccessible directly from the public Internet and does not require router port forwarding.

## Result

The homelab can now be accessed remotely through an authenticated Tailscale connection while internal services remain isolated from direct Internet exposure.

Vaultwarden was successfully tested over cellular connectivity using its existing `https://vault.home.arpa` address.
