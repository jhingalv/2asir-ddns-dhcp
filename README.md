# Dynamic DNS (DDNS) with BIND9 and ISC-DHCP-Server using Vagrant and Ansible

## Description
This project implements **Dynamic DNS (DDNS)** in a local network using **BIND9** and **ISC-DHCP-Server**, fully automated with **Vagrant** and **Ansible**.

DDNS allows DHCP clients to automatically update their DNS records (A and PTR) whenever their IP addresses change, ensuring hostnames always resolve correctly without manual intervention.

## Network Scenario
- Subnet: `192.168.58.0/24`
- DNS Server: `192.168.58.10`
- DHCP Server: `192.168.58.20`
- Client(s): receive IP dynamically via DHCP

## Technology Stack
- **BIND9**: DNS server  
- **ISC-DHCP-Server**: DHCP server  
- **TSIG**: secure authentication for dynamic updates  
- **Vagrant**: VM provisioning  
- **Ansible**: configuration automation  
- **Debian Bookworm**: base OS for all VMs  
- Useful commands: `dig`, `nslookup`, `tsig-keygen`, `named-checkconf`, `named-checkzone`  

## File Structure

```
.
├── Vagrantfile
├── ansible
│   ├── inventory
│   ├── dns
│   │   ├── dnsprovision.yml
│   │   └── files
│   │   │   ├── named.conf.local
│   │   │   ├── named.conf.options
│   │   │   ├── db.jhingal.izv
│   │   │   └── db.192
│   ├── dhcp
│   │   ├── dhcpprovision.yml
│   │   └── files
│   │       └── dhcpd.conf
│   └── client
│       └── clientprovision.yml
├── README.md
└── .gitignore

````

## Vagrant & Ansible Setup

### 1. Launch VMs
```bash
vagrant up
````

* VMs are automatically provisioned:

  * **dns** → BIND9 configured with TSIG and forward/reverse zones
  * **dhcp** → ISC-DHCP-Server configured to send DDNS updates to DNS
  * **client** → DHCP client ready to request an IP and update DNS

### 2. Ansible Roles & Playbooks

* **dnsprovision.yml**:

  * Installs BIND9
  * Generates TSIG key
  * Configures named.conf.options, named.conf.local
  * Copies forward/reverse zone files
  * Starts and enables BIND9 service
* **dhcpprovision.yml**:

  * Installs ISC-DHCP-Server
  * Copies TSIG key from DNS
  * Configures `dhcpd.conf` for DDNS updates
  * Starts and enables DHCP service
* **clientprovision.yml**:

  * Installs `dnsutils` and DHCP client
  * Finds the correct network interface
  * Renews DHCP lease to trigger DDNS updates

## DNS Server Configuration (BIND9)

* Forward zone: `jhingal.izv` → `/etc/bind/db.jhingal.izv`
* Reverse zone: `58.168.192.in-addr.arpa` → `/etc/bind/db.192`
* TSIG key included in `named.conf.options` for secure dynamic updates.
* Example forward record:

```
dns  IN  A   192.168.58.10
```

## DHCP Server Configuration (ISC-DHCP-Server)

* Interface: `eth1`
* Lease range: `192.168.58.100 – 192.168.58.200`
* Configured for DDNS updates to DNS server with TSIG authentication:

```
zone jhingal.izv. {
    primary 192.168.58.10;
    key "ddns-key";
}

zone 58.168.192.in-addr.arpa. {
    primary 192.168.58.10;
    key "ddns-key";
}
```

## Client Configuration

* Installs `dnsutils` and DHCP client
* Identifies the interface with `192.168.58.x`
* Renews DHCP lease to receive an IP and update DNS automatically

## Testing DDNS

1. **Renew the DHCP lease on the client VM**

  First, identify the network interface connected to the 192.168.58.0/24 network:

  ```bash
  ip a
  ```

  Look for the interface with an IP in the 192.168.58.x range (usually `eth1` in this Vagrant setup). Then release and renew the DHCP lease:

  ```bash
  sudo dhclient -r eth1   # Replace 'eth1' with your interface if different
  sudo dhclient eth1
  ```

2. **Verify that the DNS records have been updated**

  Use `dig` or `nslookup` to check both forward and reverse DNS resolution:

  ```bash
  # Forward lookup
  dig dns.jhingal.izv
  nslookup dns.jhingal.izv

  # Reverse lookup (PTR record)
  dig -x 192.168.58.X
   ```

  Replace `192.168.58.X` with the IP assigned to the client.

3. **Check logs for confirmation**

  You can verify that the DHCP and DNS servers processed the updates correctly by checking their logs:

  * DHCP server logs: `/var/log/syslog`
  * DNS server logs: `/var/log/syslog` or `/var/log/daemon.log` (if BIND9 logging is customized)

## Author

* Amador Hinojosa Gálvez