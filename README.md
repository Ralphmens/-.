# SRWE Final PT Skills Assessment (PTSA) - Lab Guide

## Overview

This lab involves configuring a small network with:

- 1 Router (R1)
- 2 Switches (S1, S2)
- 2 PCs (PC-A, PC-B)
- Both IPv4 and IPv6 connectivity

**Important:** All configurations must be done in PT Physical Mode using direct console connections (not logical topology view).

---

## PART 1: Build the Network

### Task: Physical Setup

**Question:** How do I set up the physical network topology?

**Answer:**

1. **Place devices on rack/table:**
   - Move R1 to the equipment rack
   - Move S1 to the equipment rack
   - Move S2 to the equipment rack
   - Place PC-A on the table
   - Place PC-B on the table

2. **Cable connections (use Copper Straight-Through):**
   - R1 G0/1 → S1 F0/5
   - S1 F0/1 → S2 F0/1
   - S1 F0/2 → S2 F0/2
   - S1 F0/6 → PC-A FastEthernet0
   - S2 F0/18 → PC-B FastEthernet0

3. **Power on all devices:**
   - Turn on PC-A (click the ON button)
   - Turn on PC-B
   - Turn on R1

4. **Connect console cables for configuration:**
   - Console cable from R1 Console port → PC-A RS232 port
   - Console cable from S1 Console port → PC-B RS232 port
   - (Later you'll reconnect from S1 to S2 when needed)

---

## PART 2: Configure Initial Device Settings

### Step 1: Configure R1 Basic Settings

#### A. Basic Configuration

**Question:** What basic settings need to be configured on R1?

**Answer:**
Access R1 via PC-A Terminal, then:

```
enable
configure terminal
no ip domain-lookup
hostname R1
banner motd # Warning unauthorized access is prohibited #
```

**Explanation:**

- `no ip domain-lookup` - Prevents router from trying to resolve mistyped commands as domain names
- `hostname R1` - Sets the hostname
- `banner motd` - Sets the Message of the Day warning banner

#### B. Password Security

**Question:** How do I configure password security on R1?

**Answer:**

```
line console 0
password conpassword
login
exit

enable secret enpassword

service password-encryption

security passwords min-length 10
```

**Explanation:**

- Console password: `conpassword`
- Enable secret: `enpassword`
- `service password-encryption` - Encrypts all clear text passwords
- Minimum password length: 10 characters

#### C. SSH Configuration

**Question:** How do I configure SSH access on R1?

**Answer:**

```
username admin secret admin1pass

ip domain-name CCNA-ptsa.com

crypto key generate rsa
[When prompted for modulus: 1024]

ip ssh version 2

line vty 0 15
login local
transport input ssh
```

**Explanation:**

- Creates local admin user with password `admin1pass`
- Sets domain name to `CCNA-ptsa.com`
- Generates RSA key with 1024-bit modulus
- Forces SSH version 2 (more secure)
- VTY lines authenticate against local database and only accept SSH

---

### Step 2: Configure Router Interfaces

#### A. Loopback Interface

**Question:** How do I configure the loopback interface on R1?

**Answer:**
Check the addressing table for loopback IP addresses:

- IPv4: 209.165.200.1/27
- IPv6: 2001:db8:acad:209::1/64
- Link-local: FE80::1

```
interface loopback 0
description Loopback interface
ip address 209.165.200.1 255.255.255.224
ipv6 address 2001:db8:acad:209::1/64
ipv6 address FE80::1 link-local
```

#### B. Router Sub-interfaces

**Question:** How do I configure sub-interfaces for VLANs on R1?

**Answer:**
First, enable IPv6 routing:

```
ipv6 unicast-routing
```

Then configure each sub-interface according to the VLAN and addressing tables:

**VLAN 2 (Bikes) - G0/1.2:**

```
interface g0/1.2
description Bikes
encapsulation dot1q 2
ip address 10.19.8.1 255.255.255.192
ipv6 address 2001:db8:acad:a::1/64
ipv6 address FE80::1 link-local
```

**VLAN 3 (Trikes) - G0/1.3:**

```
interface g0/1.3
description Trikes
encapsulation dot1q 3
ip address 10.19.8.65 255.255.255.224
ipv6 address 2001:db8:acad:b::1/64
ipv6 address FE80::1 link-local
```

**VLAN 4 (Management) - G0/1.4:**

```
interface g0/1.4
description Management
encapsulation dot1q 4
ip address 10.19.8.97 255.255.255.248
ipv6 address 2001:db8:acad:c::1/64
ipv6 address FE80::1 link-local
```

**VLAN 6 (Native) - G0/1.6:**

```
interface g0/1.6
description Native
encapsulation dot1q 6 native
```

**Activate the physical interface:**

```
interface g0/1
no shutdown
```

**Explanation:**

- Each sub-interface uses dot1q encapsulation with its VLAN number
- VLAN 6 is marked as `native` (untagged VLAN)
- All use FE80::1 as link-local IPv6 address
- Don't forget to enable the physical interface G0/1!

---

### Step 3: Configure S1

#### A. Basic Settings

**Question:** What basic settings need to be configured on S1?

**Answer:**
Access S1 via PC-B Terminal:

```
enable
configure terminal
no ip domain-lookup
hostname S1
banner motd # Warning unauthorized access is prohibited #
```

#### B. Password Security

**Question:** How do I configure passwords on S1?

**Answer:**

```
line console 0
password conpassword
login
exit

enable secret enpassword

service password-encryption

security passwords min-length 10
```

#### C. SSH Configuration

**Question:** How do I configure SSH on S1?

**Answer:**

```
username admin secret admin1pass

ip domain-name CCNA-ptsa.com

crypto key generate rsa
[When prompted: 1024]

ip ssh version 2

line vty 0 15
login local
transport input ssh
```

#### D. Management Interface

**Question:** How do I configure the management interface on S1?

**Answer:**
From the addressing table, S1 uses VLAN 4 with IP 10.19.8.98/29

```
interface vlan 4
ip address 10.19.8.98 255.255.255.248
ipv6 address 2001:db8:acad:c::98/64
ipv6 address FE80::98 link-local
no shutdown
exit

ip default-gateway 10.19.8.97
```

**Explanation:**

- VLAN 4 is the management VLAN
- Default gateway points to R1's G0/1.4 interface

---

### Step 4: Configure S2

#### A. Basic Settings

**Question:** What basic settings need to be configured on S2?

**Answer:**
Reconnect console cable from S1 to S2, then access S2 terminal:

```
enable
configure terminal
no ip domain-lookup
hostname S2
banner motd # Warning unauthorized access is prohibited #
```

#### B. Password Security

```
line console 0
password conpassword
login
exit

enable secret enpassword

service password-encryption

security passwords min-length 10
```

#### C. SSH Configuration

```
username admin secret admin1pass

ip domain-name CCNA-ptsa.com

crypto key generate rsa
[When prompted: 1024]

ip ssh version 2

line vty 0 15
login local
transport input ssh
```

#### D. Management Interface

**Question:** How do I configure the management interface on S2?

**Answer:**
From the addressing table, S2 uses VLAN 4 with IP 10.19.8.99/29

```
interface vlan 4
ip address 10.19.8.99 255.255.255.248
ipv6 address 2001:db8:acad:c::99/64
ipv6 address FE80::99 link-local
no shutdown
exit

ip default-gateway 10.19.8.97
```

---

## PART 3: Configure Network Infrastructure Settings

### Step 1: Configure VLANs on Both Switches

**Question:** How do I create VLANs on S1 and S2?

**Answer:**
On both S1 and S2:

```
vlan 2
name Bikes
exit

vlan 3
name Trikes
exit

vlan 4
name Management
exit

vlan 5
name Parking_Lot
exit

vlan 6
name Native
exit
```

**Explanation:**

- VLAN 2: Bikes
- VLAN 3: Trikes
- VLAN 4: Management
- VLAN 5: Parking_Lot (for unused ports)
- VLAN 6: Native VLAN

---

### Step 2: Configure EtherChannel (Port Channel)

**Question:** How do I configure EtherChannel between S1 and S2?

**Answer:**

**On S1:**

```
interface range f0/1-2
channel-group 1 mode active
exit

interface port-channel 1
switchport mode trunk
switchport trunk native vlan 6
switchport trunk allowed vlan 2,3,4,6
no shutdown
```

**On S2:**

```
interface range f0/1-2
channel-group 1 mode active
exit

interface port-channel 1
switchport mode trunk
switchport trunk native vlan 6
switchport trunk allowed vlan 2,3,4,6
no shutdown
```

**Verification:**

```
show etherchannel summary
show interfaces trunk
```

**Explanation:**

- Uses LACP (Link Aggregation Control Protocol) with `mode active`
- Bundles F0/1 and F0/2 into Port-Channel 1
- Configured as trunk with native VLAN 6
- Allows VLANs 2, 3, 4, and 6

---

### Step 3: Configure Switch Ports

#### A. Access Ports for PCs

**Question:** How do I configure the ports connected to PCs?

**Answer:**

**On S1 (PC-A connected to F0/6):**

```
interface f0/6
description Connected to PC-A
switchport mode access
switchport access vlan 2
```

**On S2 (PC-B connected to F0/18):**

```
interface f0/18
description Connected to PC-B
switchport mode access
switchport access vlan 3
```

#### B. Port Security

**Question:** How do I configure port security on the access ports?

**Answer:**

**On S1:**

```
interface f0/6
switchport port-security
switchport port-security maximum 3
```

**On S2:**

```
interface f0/18
switchport port-security
switchport port-security maximum 3
```

**Explanation:**

- Enables port security
- Limits to maximum 3 MAC addresses per port

#### C. Trunk Port to Router

**Question:** How do I configure the trunk port connecting to R1?

**Answer:**

**On S1 (F0/5 connects to R1):**

```
interface f0/5
switchport mode trunk
switchport trunk native vlan 6
switchport trunk allowed vlan 2,3,4,6
```

#### D. Unused Ports

**Question:** How do I secure unused ports?

**Answer:**

**On S1:**
First check which ports are unused:

```
do show vlan brief
```

Then configure unused ports (F0/3-4, F0/7-24, G0/1-2):

```
interface range f0/3-4
switchport mode access
switchport access vlan 5
description Unused ports
shutdown
exit

interface range f0/7-24
switchport mode access
switchport access vlan 5
description Unused ports
shutdown
exit

interface range g0/1-2
switchport mode access
switchport access vlan 5
description Unused ports
shutdown
```

**On S2:**
Configure unused ports (F0/3-17, F0/19-24, G0/2):

```
interface range f0/3-17
switchport mode access
switchport access vlan 5
description Unused port
shutdown
exit

interface range f0/19-24
switchport mode access
switchport access vlan 5
description Unused port
shutdown
exit

interface range g0/2
switchport mode access
switchport access vlan 5
description Unused port
shutdown
```

**Explanation:**

- All unused ports assigned to VLAN 5 (Parking_Lot)
- Ports are shut down for security
- Descriptive label helps identify their status

---

## PART 4: Configure Host Support

### Step 1: Configure Default Routing on R1

**Question:** How do I configure default routes on R1?

**Answer:**

```
ip route 0.0.0.0 0.0.0.0 loopback 0

ipv6 route ::/0 loopback 0
```

**Explanation:**

- IPv4 default route uses loopback 0 as exit interface
- IPv6 default route uses loopback 0 as exit interface

---

### Step 2: Configure DHCP for VLAN 2 (Bikes)

**Question:** How do I configure DHCP for VLAN 2?

**Answer:**

From the addressing table:

- VLAN 2 subnet: 10.19.8.0/26
- Use only the last 10 host addresses
- Default gateway: 10.19.8.1

```
ip dhcp excluded-address 10.19.8.1 10.19.8.52

ip dhcp pool CCNA-A
network 10.19.8.0 255.255.255.192
default-router 10.19.8.1
domain-name CCNA-ptsa.net
```

**Explanation:**

- Excludes addresses .1 through .52, leaving last 10 addresses (.53-.62) for DHCP
- Pool named CCNA-A
- Default gateway is R1's interface

---

### Step 3: Configure DHCP for VLAN 3 (Trikes)

**Question:** How do I configure DHCP for VLAN 3?

**Answer:**

From the addressing table:

- VLAN 3 subnet: 10.19.8.64/27
- Use only the last 10 host addresses
- Default gateway: 10.19.8.65

```
ip dhcp excluded-address 10.19.8.65 10.19.8.84

ip dhcp pool CCNA-B
network 10.19.8.64 255.255.255.224
default-router 10.19.8.65
domain-name CCNA-B.NET
```

**Explanation:**

- Excludes addresses .65 through .84, leaving last 10 addresses (.85-.94) for DHCP
- Pool named CCNA-B
- Default gateway is R1's interface

Don't forget to save:

```
copy running-config startup-config
```

---

### Step 4: Configure Host Computers

**Question:** How do I configure PC-A and PC-B?

**Answer:**

**PC-A:**

1. Close terminal
2. Go to IP Configuration
3. Select DHCP for IPv4 (will get address from CCNA-A pool)
4. Configure IPv6 manually:
   - IPv6 Address: 2001:db8:acad:a::a/64
   - Default Gateway: FE80::1

**PC-B:**

1. Close terminal
2. Go to IP Configuration
3. Select DHCP for IPv4 (will get address from CCNA-B pool)
4. Configure IPv6 manually:
   - IPv6 Address: 2001:db8:acad:b::b/64
   - Default Gateway: FE80::1

---

## Verification and Testing

**Question:** How do I verify the configuration is working?

**Answer:**

**Test connectivity from PC-A:**

1. Open Command Prompt
2. Get PC-B's IP address (or use your knowledge from DHCP pool)
3. Test IPv4 ping:

   ```
   ping [PC-B IPv4 address]
   ```

   _Note: You may get 1-2 timeouts initially due to ARP/convergence delay_

4. Test IPv6 ping:

   ```
   ping 2001:db8:acad:b::b
   ```

5. Test default gateway:

   ```
   ping 10.19.8.1
   ```

6. Test switch management IPs:
   ```
   ping 10.19.8.98 (S1)
   ping 10.19.8.99 (S2)
   ```

**Expected Result:** All pings should eventually succeed (after initial 1-2 timeouts). If you get 100% completion status in Packet Tracer, your configuration is correct!

---

## Quick Reference - Key IP Addresses

| Device | Interface | IPv4 Address     | IPv6 Address            | VLAN |
| ------ | --------- | ---------------- | ----------------------- | ---- |
| R1     | Loopback0 | 209.165.200.1/27 | 2001:db8:acad:209::1/64 | -    |
| R1     | G0/1.2    | 10.19.8.1/26     | 2001:db8:acad:a::1/64   | 2    |
| R1     | G0/1.3    | 10.19.8.65/27    | 2001:db8:acad:b::1/64   | 3    |
| R1     | G0/1.4    | 10.19.8.97/29    | 2001:db8:acad:c::1/64   | 4    |
| S1     | VLAN 4    | 10.19.8.98/29    | 2001:db8:acad:c::98/64  | 4    |
| S2     | VLAN 4    | 10.19.8.99/29    | 2001:db8:acad:c::99/64  | 4    |
| PC-A   | NIC       | DHCP (VLAN 2)    | 2001:db8:acad:a::a/64   | 2    |
| PC-B   | NIC       | DHCP (VLAN 3)    | 2001:db8:acad:b::b/64   | 3    |

## Key Passwords

- Console password: `conpassword`
- Enable secret: `enpassword`
- SSH username: `admin`
- SSH password: `admin1pass`

---

**End of Lab Guide**
