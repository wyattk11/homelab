# Cyber Range Reconnaissance and Vulnerability Assessment

Rebuilt the reconnaissance phase of my IS 4543 capstone inside the isolated cyber range. The original version scanned an AWS EC2 instance. Doing it again from Kali against local targets let me revisit each tool step by step, and practice separating real findings from scanner noise.

Learning goals:
- Understand what each Nmap option adds, rather than running one long command
- Use version information to find leads, then confirm or rule them out
- Find configuration weaknesses that version-based scanning cannot see
- Write findings with evidence, severity and remediation

## Environment

```
Kali Linux (VM 200)  10.10.10.102   attacker
target-web (VM 202)  10.10.10.50    Ubuntu 22.04, Apache, vsftpd, OpenSSH
target-msf (VM 203)  10.10.10.154   Metasploitable 2
```

All testing stayed inside `10.10.10.0/24`, behind the OPNsense firewall. Both targets were restored from their `clean-build` snapshots before testing.

## Methodology

### Host Discovery and Port Scanning

```
nmap 10.10.10.50 10.10.10.154
```

- Default scan of the top 1,000 TCP ports
- target-web: 3 open ports (21, 22, 80)
- target-msf: 23 open ports
- Closed ports first showed as `filtered` because of a Proxmox firewall issue in the lab, not on the targets. The fix is documented in `infrastructure/cyber-range`

### Service and OS Detection

```
sudo nmap -sV -O 10.10.10.50 10.10.10.154 -oA quick-sv
```

- `-sV` identified software versions, for example vsftpd 3.0.5 on target-web and vsftpd 2.3.4 on target-msf
- `-O` fingerprinted the operating system: Linux 5.x on target-web and Linux 2.6 on target-msf
- `-oA` saved the results in normal, grepable and XML formats for later parsing

### Vulnerability Lookup

```
searchsploit --nmap quick-sv.xml
sudo nmap -sV --script vulners <target> -oA <name>
```

- `searchsploit` matched service names against an offline copy of Exploit-DB
- The `vulners` NSE script matched each detected version against CVE data, with CVSS scores

### Configuration Checks

```
sudo nmap -sC -sV -p 21,22,80 10.10.10.50 -oA web-scripts
curl -I http://10.10.10.50/
nikto -h http://10.10.10.50 -o nikto-web.txt
```

- Default NSE scripts tested service behavior, for example whether anonymous FTP login works
- `curl -I` showed the HTTP response headers
- Nikto checked the web server for missing headers, risky methods and known issues

### Credentialed Verification

- Logged in to target-web over SSH with the weak test account
- Checked the installed package version and Ubuntu's changelog for specific CVEs, to confirm or rule out version-based findings

## Findings: target-web

1. **Weak credentials over SSH (High)**
   - Evidence: logged in as the test account with a trivial password
   - Fix: enforce strong passwords, use key-only SSH, add fail2ban
2. **SSH password authentication enabled (Medium)**
   - Evidence: the same password login succeeded
   - Fix: set `PasswordAuthentication no`
3. **Anonymous FTP access (Medium to High)**
   - Evidence: `ftp-anon` returned FTP code 230 (login successful)
   - Fix: set `anonymous_enable=NO`
4. **Cleartext FTP (Medium)**
   - Evidence: `ftp-syst` reported that the control and data connections are plain text
   - Fix: disable FTP and use SFTP
5. **Missing security updates (Medium)**
   - Evidence: the login banner listed 19 pending security updates
   - Fix: apply updates and enable unattended-upgrades
6. **No HTTPS (Medium)**
   - Evidence: only port 80 was open, and no TLS service was offered
   - Fix: serve the site over HTTPS and redirect HTTP to it
7. **Missing HTTP security headers (Low)**
   - Evidence: `curl -I` and Nikto both showed no Content-Security-Policy, X-Content-Type-Options, Referrer-Policy or Permissions-Policy headers
   - Fix: add the headers with `Header always set` in Apache
8. **Server version disclosure (Low)**
   - Evidence: the `Server` header revealed `Apache/2.4.52 (Ubuntu)`
   - Fix: set `ServerTokens Prod`

### Ruled Out

- **CVE-2024-6387 (regreSSHion): false positive**
  - The `vulners` script flagged OpenSSH 8.9p1 as vulnerable, based only on the upstream version number
  - The credentialed check showed package `1:8.9p1-3ubuntu0.18`, and Ubuntu's changelog references CVE-2024-6387, which means the fix was backported
- **Nikto "Apache appears outdated"**: same reason. Ubuntu backports security fixes without changing the version number
- **Allowed HTTP methods**: only GET, POST, OPTIONS and HEAD were allowed. No PUT, DELETE or TRACE

## Findings: target-msf

Metasploitable 2 is vulnerable by design, so I focused on the highest-impact issues:

- **Bindshell on port 1524 (Critical)**: gives an unauthenticated root shell
- **vsftpd 2.3.4 backdoor, CVE-2011-2523 (Critical)**: an exact version match, flagged by both searchsploit and vulners, with a public Metasploit module
- **UnrealIRCd backdoor, CVE-2010-2075 (Critical)**: found by searchsploit using the product name
- **Samba "usermap script", CVE-2007-2447 (Critical)**: missed by both lookup tools, because Nmap only reported "3.X - 4.X"
- **Telnet and r-services on ports 23 and 512-514 (High)**: cleartext authentication by design
- **End-of-life software throughout (High)**: OpenSSH 4.7, Apache 2.2.8, MySQL 5.0, PostgreSQL 8.3 and Linux 2.6

## Lessons Learned

- A version number is a lead, not a finding. On distributions that backport patches, unauthenticated version matching produces many false positives, and a credentialed check resolves them
- Automated lookups miss some of the most serious issues. Services with no detectable version, such as the bindshell and Samba, and design flaws such as cleartext protocols, never matched a CVE
- Scanner output needs triage. `vulners` returned hundreds of entries, including duplicate IDs for the same CVE, CVEs for newer versions matched through loose version ranges, and client-side bugs listed against a server
- Configuration testing (`-sC`, Nikto, logging in) found most of the real issues on the patched host

## Next Steps

These lessons are the design requirements for v2 of my report generator:

- Parse Nmap XML for every host and port, instead of using hardcoded filenames
- Look up exploits and CVEs per service, and remove duplicate CVE IDs
- Rank findings by exploit availability and CVSS score, instead of keyword matching
- Label each finding's confidence: exploit confirmed, version match only, or manual review needed (no version detected)
- Add checks that do not depend on CVEs, such as cleartext protocols, anonymous FTP and missing headers
- Write reports to timestamped folders, one section per host
