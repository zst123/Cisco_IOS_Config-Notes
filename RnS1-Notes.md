# CCNA R&S 1 Notes

--------------------------------------------------------------------------

## Configuring Initial Switch Settings

### Enter privileged EXEC mode
	Switch> enable
	Switch#
 
### Examine RAM configuration (running-configuration)
	Switch# show running-config
	Switch# show run    // Alternatively
 
### Assign hostname
	Switch# config t
	Switch(config)# hostname S1
	
### MOTD Banner
	S1(config)# banner motd "This is a secure system. Authorized Access Only!"
***Banner is displayed on first login (console line 0)***

### Disable DNS lookup
	S1(config-if)# no ip domain-lookup

### Console line password (Secure access to the console line)
	Switch# config t
	Switch(config)# line console 0
	Switch(config-line)# password cisco
	Switch(config-line)# login
	Switch(config-line)# exit
 
### VTY password (Secure access to the virtual terminal)
	S1(config)# line vty 0
	S1(config)# line vty 0 4
	S1(config)# line vty 0 15    // For lines 0 to 15
	S1(config-vty)# password cisco
	S1(config-vty)# login
 
### Privileged mode password or encrypted password/secret (Secure privileged mode access)
	S1# config t
	S1(config)# enable password cisco
	S1(config)# enable secret itsasecret
***Note: The enable secret password overrides the enable password. If both are configured on the switch, you must enter the enable secret password to enter privileged EXEC mode.***

---
 
## Configure Security

### Encrypt passwords, min-length passwords
	S1# config t
	S1(config)# service password-encryption
	S1(config)# security password min-length 8
	
### Block brute force attacks
	S1(config)# login block-for 120 attempts 3 within 60
	
### Set EXEC timeout for config lines
	S1(config)# line vty 0 4
	S1(config-vty)# exec-timeout 10

## Configuring SSH

**Set up SSH on Router:**
> - Set the domain name
> - Generate a 1024-bit RSA key.
> - Configure the VTY lines for SSH access.
> - Use the local user profiles for authentication.
> - Create a username & password

**Use SSH (PC and Router): **
> `SSH -l username target`

### Domain Name
	R1(config)#ip domain name MyRouter
	
### Generate 1024-bit RSA keys
	R1(config)# crypto key generate rsa
	How many bits in the modulus [512]: 1024

### Set SSH as the protocol for VTY line
	R1(config)# line vty 0 4
	R1(config-line)#no transport input all    // disable Telnet
	R1(config-line)#transport input ssh    // enable SSH 
	R1(config-line)#login local
	R1(config)#ip ssh version 2

### Use local user profiles for authentication (Local Password Checking)
	R1(config-line)# login local

### Create username with password (for SSH)
	R1(config)# username admin password admin123
	R1(config)# username admin secret ENCRYPTEDPASSWORD
	R1(config)# username admin privilege 15 secret admin123
	// highest privilege is 15
	R1(config)# no username DELETEUSER
	

---
## Configuration Files / File System

### Save configuration files to NVRAM
	S1# copy running-config startup-config
	S1# copy r s    // Alternatively
	Destination filename [startup-config]?[Enter]

> ***Changes are made to a running configuration in `RAM` or `running-configuration` and copied to `NVRAM` or `startup-configuration`. Verify that the configuration is accurate using the `show run` command.***

### Backup running-config to TFTP Server
	S1# copy run tftp

### Copy files in flash to TFTP
	R1# copy flash tftp

### List files/config/filesystem
	R# show run
	R# show flash
	R# show file systems

### Erase files
	S1# erase startup-config    // Erase NVRAM contents

---
## Implementing Basic IPv4 Connectivity

### Set IP address
	S1# config t
	S1(config)# interface vlan 1
	S1(config)# interface GigabitEthernet 0/1
	S1(config-if)# ip address 192.168.1.253 255.255.255.0
	S1(config-if)# no shutdown    // turn on & activate port interface

### Set default gateway
	S1(config)# ip default-gateway 10.10.10.1

### Domain Name
	R1(config)#ip domain name MyRouter
	
### Document interfaces with descriptions on the router interfaces / switch virtual interface.
	Router(config)# interface vlan 1
	Router(config-if)# description this is my vlan!

### Layer 3 Switch
	S1(config-if)# no switchport    // Change port to routing mode

***For L3 capable switch, puts the interface in L3 mode and makes it operate more like a router interface rather than a switch port***

---
## Configuring IPv6 Connectivity

### Enable router to forward IPv6 packets.
	R1(config)# ipv6 unicast-routing

### Set IPv6 address / link-local address
	R1(config-if)# ipv6 address 2001:DB8:1:1::1/64
	R1(config-if)# ipv6 address FE80::1 link-local
	R1(config-if)# no shutdown


---
## Verifying Switch/Router Configuration

### Verify password (console line / privileged mode)
	Password: // line console 0 password
	S1>
	S1> enable
	Password: // privileged mode -> secret overrides password

### Verify IP address configuration
	S1# show ip interface brief
	S1# show running-config
	R1# show ip interface Serial 0/0/0

### Verify connectivity
	R1# ping 192.168.254.1
	R1# traceroute 192.168.0.1

### Verify SSH
	R1# show ip ssh   // version and settings
	R1# show ssh    // verify if SSH is currently running
	

---
## Verifying PC connectivity / addressing

### IPv4
	ping 127.0.0.1
	tracert 10.10.1.20
	ipconfig /all

### IPv6
	ping 2001:DB8:1:4::1
	tracert 2001:DB8:1:4::A
	ipv6config /all

---
## Misc
### Check time. Set time
    Switch# show clock
	Switch# clock set 15:00:00 30 Apr 2035

---
## Cabling

### Straight-through cable
- PC - Hub/Switch
- Hub/Switch - Router

### Crossover cable
- Hub/Switch - Hub/Switch
- Router - Router
- PC - PC
- ***PC - Router***

### Clock Rate for DCE interface
    Router(config)# int Serial1/1
    Router(config-if)# clock rate 128000

***A Serial Cable has a DCE and a DTE side. The DCE side is elected as the Clock Master, so the device which has the DCE side of the cable connected to it must be the one which you enter the clock rate command.***

- DCE devices: Modems 
- DTE devices: PC, NIC

---
## IP Subnetting

### Subnetting a Class C network
| Borrow Bits | CIDR Notation | Subnet Mask | Usable Hosts | Subnetworks |
| :-: | :-: | :- | :-: | :-: |
| 0 | /24 | 255.255.255.0   | 256 - 2 | 1  |
| 1 | /25 | 255.255.255.128 | 128 - 2 | 2  |
| 2 | /26 | 255.255.255.192 |  64 - 2 | 4  |
| 3 | /27 | 255.255.255.224 |  32 - 2 | 8  |
| 4 | /28 | 255.255.255.240 |  16 - 2 | 16 |
| 5 | /29 | 255.255.255.248 |   8 - 2 | 32 |
| 6 | /30 | 255.255.255.252 |   4 - 2 | 64 |
