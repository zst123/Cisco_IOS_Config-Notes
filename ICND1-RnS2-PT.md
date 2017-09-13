# ICND1 R&S 2 - IOS Configurations

--------------------------------------------------------------------------

## Routing
    show ip route
    show ip route static
    show ipv6 route

### IPv4 Static Routing
    Router(config)# ip route <final-dest-network> <mask> <next-hop-router-ip/interface>

    ! Directly Attached Static Routes
    ip route 192.168.1.0 255.255.255.0 s0/1

    ! Recursive Static Routes
    ip route 192.168.1.0 255.255.255.0 120.1.1.22

    ! Fully Specified Static Routes
    ip route 192.168.1.0 255.255.255.0 s0/1 120.1.1.22

### IPv6 Static Routing
    ! Directly Attached Static Routes
    ipv6 route 2001:DB8::/32 G0/0
    
    ! Recursive Static Routes
    ipv6 route 2001:DB8::/32 2001:DB8:3000:1
    
    ! Fully Specified Static Routes
    ipv6 route 2001:DB8::/32 G0/0 2001:DB8:3000:1

### Default Static Routing
    ip route 0.0.0.0 0.0.0.0 <next-hop-ip>
    ipv6 route ::/0 <next-hop-ip>

### Floating Static Route
    ! AD = 0 (Directly Attached)
    ip route 172.16.10.0 255.255.255.0 s0/1

    ! AD = 1 (Recursive)
    ip route 172.16.10.0 255.255.255.0 10.10.10.2

    ! AD = 200 (Floating)
    ip route 172.16.10.0 255.255.255.0 10.10.10.2 200

### RIPv2
    R3(config)# router rip
    R3(config-router)# network 10.1.1.0
    R3(config-router)# version 2
    R3(config-router)# no auto-summary
    R3(config-router)# redistribute static

---

## Switchport Security
    show port-security

    switchport mode access
    switchport port-security
    switchport port-security violation shutdown 
    switchport port-security maximum 2
    switchport port-security mac-address sticky 
*Ping PCs to record mac-address*

## Inter-VLAN Routing
    show vlan brief
    show interface trunk

### VLAN (access mode)
    S1(config)# vlan 10
    S1(config-vlan)# name AdminDept

    S1(config)# int f0/1
    S1(config-if)# switchport access vlan 10
    S1(config-if)# switchport mode access

### Trunking
    switchport mode trunk

### Subinterface / Router on a Stick    
    ! Turn on interface
    int g0/0
    no shut

    ! Setup subinterface
    int g0/0.10
    encap dot1q 10
    ip add 172.17.10.1 255.255.255.0
*Since router configured with multiple subinterfaces assigned to different VLANs, switch port must be configured as a trunk*

### Native VLAN
    ! Switch
    switchport native vlan 1
    
    ! Router subinterface
    int g0/0.10
    encap dot1q 1 native

--- 

## Access Control List (ACL)
    show access-lists

* Standard ACLs: as close to dest. (Since dest address not specified)
* Extended ACLs: as close to source.

### Numbered Standard ACLs
**<center><u> 192.168.11.0/24 network is not allowed access to the WebServer on the 192.168.20.0/24 network </u></center>**

    ! Deny from 192.168.11.0
    R2(config)# access-list 1 deny 192.168.11.0 0.0.0.255 
    R2(config)# access-list 1 permit any

    ! Traffic flows out from router to 192.168.11.0 interface
    R2(config)# interface GigabitEthernet0/0
    R2(config-if)# ip access-group 1 out

### Named Standard ACLs

    R1(config)# ip access-list standard File_Server_Restrictions
    R1(config-std-nacl)# permit host 192.168.20.4
    R1(config-std-nacl)# deny any

    R1(config)# int fa0/0
    R1(config-if)# ip access-group File_Server_Restrictions out


---

## DHCP
    show ip dhcp pool|binding|conflict

### Exclude IP range
    ip dhcp excluded-address 192.168.10.1 192.168.10.10

### DHCP Pool
    R2(config)# ip dhcp pool R1-LAN
    R2(dhcp-config)# network 192.168.10.0 255.255.255.0    // LAN network address
    R2(dhcp-config)# default-router 192.168.10.1    // LAN default gateway
    R2(dhcp-config)# dns-server 209.165.201.14

### DHCP Relay Agent / DHCP Helper
    R1(config)# int g0/0    // LAN interface
    R1(config-if)# ip helper-address 10.1.1.2    // DHCP server or router interface

### DHCP Client
    ip address dhcp 
    no shut

### Disable DHCP server
    no service dhcp

---

## IPv6 

### SLAAC (Stateless Autoconfiguration)
    Device(config-if)# ipv6 address autoconfig

---

## NTP network / Software clock

### Stratum level
- stratum level = hop counts from authoritative time source (stratum 0)
- Max hop count is 15
- device unsynchronized = stratum 16 

### Set time manually
    R# clock set 15:00:00 30 Apr 2035

### NTP Server
    R(config)# ntp server 209.165.200.225

### Verify NTP (if device is synchronized)
    show ntp status

### Verify Time Source
    show clock detail
***`Time source is user configuration` / `Time source is NTP`***

---

### Password Recovery

*Access ROMMON mode using break sequence (Ctrl+Break) during boot up / removing external flash memory*

    ! ignore startup config & reboot device
    rommon 1 > confreg 0x2142
    rommon 2 > reset

    ! change password
    Router# copy start run
    Router# conf t
    Router(config)# enable secret cisco
    Router(config)# config-register 0x2102
    Router(config)# end
    Router# copy running-config startup-config

---

## CDP / LLDP

### Cisco Discovery Protocol (CDP)
> a Cisco proprietary Layer 2 protocol that is used to gather information about Cisco devices which share the same data link. 

    ! Verify
    Router# show cdp
    Router# show cdp interface
    Router# show cdp neighbors
    Router# show cdp neighbors details

    ! For interface
    Switch(config)# int G0/0
    Switch(config-if)# cdp enable
    Switch(config-if)# no cdp enable

    ! Globally
    Router(config)# cdp run
    Router(config)# no cdp run

### Link Layer Discovery Protocol (LLDP)
> a vendor neutral neighbor discovery protocol similar to CDP

    ! Verify
    Switch# show lldp
    Switch# show lldp neighbors
    Switch# show lldp neighbors details

    ! For interface
    Switch(config)# int G0/0
    Switch(config-if)# lldp transmit
    Switch(config-if)# lldp receive

    ! Globally
    Switch(config)# lldp run

> Both CDP and LLDP can gather information from directly connected network devices, such as routers, switches, and wireless APs. Also, they can both collect Layer 3 IP addresses of neighboring devices. When configured at the interface level, LLDP is more customizable than CDP because LLDP can be configured to only transmit or receive LLDP packets.

---

## Others

### SSHv2
    line vty 0
    no transport input all
    transport input ssh
    login local
    ip ssh version 2
