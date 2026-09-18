# Minecraft Server

Deployed a dedicated modded Minecraft server in my Proxmox homelab to gain additional experience with Linux server administration, virtualization, networking, Java applications, resource management, and systemd services.

The server currently hosts **Better Fantasy Multiverse v1.4** using NeoForge and is accessible to clients on the local network.

## Deployment

Created a dedicated Ubuntu Server virtual machine in Proxmox for hosting Minecraft.

### Virtual Machine Resources

- Ubuntu Server 26.04
- 4 vCPU
- 8 GB RAM
- 64 GB virtual disk
- VirtIO network adapter connected to the Proxmox `vmbr0` bridge
- QEMU Guest Agent enabled
- DHCP reservation configured for `192.168.1.145`

The VM was intentionally configured without automatic startup at the Proxmox level so resource-intensive cybersecurity lab VMs and the Minecraft server can be managed independently on the 16 GB homelab host.

## Linux and SSH Configuration

After installing Ubuntu Server, the system was updated and configured for remote administration.

- Installed and enabled OpenSSH
- Installed the QEMU Guest Agent for Proxmox integration
- Configured Ed25519 public-key SSH authentication
- Verified key-based authentication before disabling password authentication
- Disabled SSH password authentication
- Disabled direct root SSH login
- Validated the SSH configuration before reloading the service

This allows the server to be administered remotely without relying on password-based SSH authentication.

## Vanilla Minecraft Testing

Before deploying the modpack, a vanilla Minecraft Java Edition server was installed and tested.

This provided a known-good baseline for validating:

- Java functionality
- Local network connectivity
- Minecraft server networking on TCP port 25565
- Client-to-server connectivity
- Whitelist functionality
- Server operator permissions

A Proxmox snapshot was created after successfully connecting to the vanilla server, providing a known-good recovery point before beginning the modded deployment.

## Better Fantasy Deployment

After validating the vanilla server, the server was migrated to **Better Fantasy Multiverse v1.4**.

The official CurseForge server pack was used rather than manually recreating the modpack configuration.

The deployment includes:

- Minecraft 1.21.1
- NeoForge 21.1.247
- Better Fantasy Multiverse v1.4
- Modpack-provided configuration files and datapacks
- Whitelist-based player access

The original vanilla server files were retained separately as an additional recovery option.

## Java Configuration

The vanilla Minecraft server required a newer Java runtime, while Better Fantasy requires Java 21.

Both Java versions were installed simultaneously:

- OpenJDK 25
- OpenJDK 21

Rather than changing the system-wide Java installation each time, the Better Fantasy startup environment was explicitly configured to use the Java 21 executable.

This allows multiple Java versions to coexist while ensuring the modded server launches with its required runtime.

## Resource Tuning

The Better Fantasy server pack originally configured the Java heap with:

    -Xms4G
    -Xmx8G

Because the VM has 8 GB of total memory, allowing Java to allocate the entire 8 GB could leave insufficient memory for Ubuntu and other system processes.

The maximum Java heap was therefore reduced to:

    -Xms4G
    -Xmx6G

This reserves approximately 2 GB of VM memory for the operating system and other overhead while still providing the modded Minecraft server with up to 6 GB of heap memory.

Initial gameplay testing showed no noticeable performance or latency issues with this configuration.

## systemd Service

The Minecraft server was initially launched manually through an SSH session for testing.

After successful testing, a dedicated `systemd` service was created to allow the server to run independently of an active SSH session.

The service:

- Runs the Minecraft server as a non-root user
- Uses `/opt/minecraft` as the working directory
- Explicitly launches Better Fantasy using Java 21
- Allows systemd to manage server startup and failures
- Starts automatically when the Minecraft VM boots

The modpack's internal automatic restart behavior was disabled so process management could be handled by systemd instead.

After deployment, the SSH session was closed and the Minecraft client successfully remained connected to the server, confirming that the service was operating independently.

## Testing and Results

The completed deployment was tested by connecting from a separate Windows gaming PC on the local network.

The following functionality was successfully verified:

- Minecraft server accessible at `192.168.1.145:25565`
- Better Fantasy Multiverse v1.4 client successfully connected
- NeoForge and modpack components loaded successfully
- World generation completed successfully
- Player whitelist enforced
- Server operator permissions functioned correctly
- Modded gameplay operated without noticeable lag
- Minecraft continued running after the administrative SSH session was closed
- systemd successfully managed the server process

## Current Architecture

    Proxmox Host
        |
        |-- VM 104 - Minecraft Server
                |
                |-- Ubuntu Server 26.04
                |-- OpenJDK 21
                |-- Better Fantasy Multiverse v1.4
                |-- NeoForge 21.1.247
                |-- systemd Minecraft Service
                |
                +-- 192.168.1.145:25565
                        |
                        +-- Local Minecraft Clients

## Future Improvements

- Configure secure access for remote players outside the home network
- Test remote multiplayer connectivity
- Implement automated Minecraft world backups
- Add backup rotation and recovery testing
- Evaluate server performance with multiple simultaneous players
- Document procedures for updating the modpack and NeoForge
