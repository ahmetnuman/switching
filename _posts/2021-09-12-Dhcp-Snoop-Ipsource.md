---
layout: post
title: DHCP Snooping and Ip Source Guard
tags: DHCP Snooping and Ip Source Guard
categories: dhcpSnooping
---

> Ahmet Numan Aytemiz, 12.09.2021

---

- **According to topology below configure DHCP snooping on the SW1 switch to prevent rogue DHCP server attacks.**
- **There is one legal DHCP server in the system which is running on the SW3-SERVER**

![Image](/img/topologysnoop.PNG)


**Tasks**

- 1. On the SW2-CLIENT swtchthes, configure ethernet 0/0 as layer 3 interface and configure it to simulate dhcp client.
- 2. On the SW1 switches configure ethernet 0/0 as a access port and assign it VLAN 1 which is default VLAN.
- 3. On the SW1 switches configure ethernet 0/1 as trunk interface with encapsulation 802.1q
- 4. On the SW3-SERVER switches configure ethernet 0/0 as trunk interface with encapsulation 802.1q
- 5. On the SW3-SERVER switches configure VLAN 1 SVI Ip address with 192.168.7.3/24 ip address
- 6. On the SW3-SERVER switches configure DHCP server with following conditions
     - DHCP Scope : 192.168.7.100-254/24
     - Default Gateway : 192.168.7.1
     - Lease Time : 7 Days
     - DHCP Excluded Address : 192.168.7.1-99 

**Tasks For DHCP Snooping:**

- 7.  Enable DHCP Snooping On the SW1  for VLAN 1  
- 8.  Configure ethernet 0/1 as a trusted interface on the SW1 
- 9.  Disable DHCP option 82 on the SW1
- 10. Renew IPv4 address of ethernet 0/0 and verify it is still working

**Tasks for IP Source Guard**

- 11. Configure linux machine static ip address with the 192.168.7.46/24 and start the ICMP traffic to 192.168.7.3
- 12. Enable IP Source Guard on the ethernet range 0/0-0/3 SW1 and verify the linux machine ICMP traffic dropped.
- 13. Configure linux machine's interface address as a dhcp client and ping to 192.168.7.3 ip address again.
- 14. Configure SW1 ethernet 0/0 as manuel ip source binding, then configure SW2-CLIENT's ethernet 0/0 interface with the static ip 192.168.7.11/24
- 15. Ensure that IP Source Guard does note block any traffic from SW2-client 

**Solutions**

- 1. On the SW2-CLIENT swtchthes, configure ethernet 0/0 as layer 3 interface and configure it to simulate dhcp client.

```
SW2-CLIENT(config)#interface Ethernet0/0
SW2-CLIENT(config-if)#description TO_SW
SW2-CLIENT(config-if)#no switchport
SW2-CLIENT(config-if)#ip address dhcp

```

---

- 2. On the SW1 switches configure ethernet 0/0 as a access port and assign it VLAN 1 which is default VLAN.

```
SW1(config)#interface Ethernet0/0
SW1(config-if)#description TO_SW-CLIENT
SW1(config-if)#switchport mode access
SW1(config-if)#switchport access vlan 1
```

---

- 3. On the SW1 switches configure ethernet 0/1 as trunk interface with encapsulation 802.1q

```
SW1(config)#interface Ethernet0/1
SW1(config-if)#description TO_SW-SERVER
SW1(config-if)#switchport trunk encapsulation dot1q
SW1(config-if)#switchport mode trunk
```

---

- 4. On the SW3-SERVER switches configure ethernet 0/0 as trunk interface with encapsulation 802.1q

```
SW3-SERVER(config)#interface Ethernet0/0
SW3-SERVER(config-if)#description TO_SW
SW3-SERVER(config-if)#switchport trunk encapsulation dot1q
SW3-SERVER(config-if)#switchport mode trunk
```

---

- 5. On the SW3-SERVER switches configure VLAN 1 SVI Ip address with 192.168.7.3/24 ip address

```
SW3-SERVER(config)#interface Vlan1
SW3-SERVER(config-if)#ip address 192.168.7.3 255.255.255.0
SW3-SERVER(config-if)#no shut
```

---

- 6. On the SW3-SERVER switches configure DHCP server with following conditions
     - DHCP Scope : 192.168.7.100-254/24
     - Default Gateway : 192.168.7.1
     - Lease Time : 7 Days
     - DHCP Excluded Address : 192.168.7.1-99

```
SW3-SERVER(config)#ip dhcp pool legalPool
SW3-SERVER(dhcp-config)#network 192.168.7.0 255.255.255.0
SW3-SERVER(dhcp-config)#default-router 192.168.7.1
SW3-SERVER(dhcp-config)#lease 7
SW3-SERVER(dhcp-config)#exit
SW3-SERVER(config)#ip dhcp excluded-address 192.168.7.1 192.168.7.99
```

---

- 7.  Enable DHCP Snooping On the SW1  for VLAN 1  

```
ip dhcp snooping
ip dhcp snooping vlan 1
```

---

- 8.  Configure ethernet 0/1 as a trusted interface on the SW1 

```
SW1(config)#interface Ethernet0/1
SW1(config-if)#ip dhcp snooping trust
```

---

- 9.  Disable DHCP option 82 on the SW1

```
SW1(config)#no ip dhcp snooping information option
```

---

- 10. Renew IPv4 address of ethernet 0/0 and verify it is still working

```
SW2-CLIENT(config)#interface ethernet 0/0
SW2-CLIENT(config-if)#no ip address
SW2-CLIENT(config-if)#ip address dhcp
SW2-CLIENT(config-if)#do show ip interface brief

```

---

- 11. Configure tiny linux machine static ip address with the 192.168.7.46/24 and start the ICMP traffic to 192.168.7.3

```
ifconfig eth0 192.168.7.46 255.255.255.0
ping 192.168.7.3
```

---

- 12. Enable IP Source Guard on the ethernet range 0/0-0/3 SW1 and verify the linux machine ICMP traffic dropped.

```
SW1(config)#interface range ethernet 0/0-3
SW1(config-if-range)#ip verify source
```

- 13. Configure tiny linux machine's interface address as a dhcp client and ping to 192.168.7.3 ip address again.

```
sudo /etc/init.d/network restart
```

- 14. Configure SW1 ethernet 0/0 as manuel ip source binding, then configure SW2-CLIENT's ethernet 0/0 interface with the static ip 192.168.7.11/24

```
SW1#show mac address-table interface ethernet 0/0
SW1#conf t
SW1(config)#ip source binding  aabb.cc00.3000 vlan 1 192.168.7.11 interface ethernet 0/0

```

---

- 15. Ensure that IP Source Guard does note block any traffic from SW2-client 

```
SW2-CLIENT(config)#interface Ethernet0/0
SW2-CLIENT(config-if)#ip address 192.168.7.99 255.255.255.0
SW2-CLIENT(config-if)#do ping 192.168.7.3
```

---

## VERIFICATION COMMANDS

```
show ip dhcp binding
show ip verify source
```
