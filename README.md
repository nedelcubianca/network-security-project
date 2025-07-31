#  Network Security Simulation Project

This project simulates a secure network architecture using **Cisco Packet Tracer**, focusing on access control, network monitoring, and offensive security testing.

##  Objective

To design and implement a realistic network scenario where:
- Devices are assigned static IPs.
- **NAT** (Network Address Translation) is configured.
- **ACLs** (Access Control Lists) are applied to restrict unauthorized access.
- **Syslog** is used to monitor and log security events.
- **Red Team techniques** are simulated to test the resilience of the network.

##  Tools & Technologies

- Cisco Packet Tracer 
- Cisco IOS commands (CLI)
- GitHub Pages for documentation

##  Project Structure

1. Network Topology
2. IP Configuration & NAT Setup
3. ACL Implementation (HTTP/HTTPS restrictions)
4. Syslog Server Monitoring
5. Red Team Simulation (port scanning, bypass attempts)
6. Results & Security Improvements

---

##  1. Network Topology

This network simulates a secure, segmented infrastructure with access control and monitoring mechanisms.

### Devices & Addresses

#### LAN 192.168.1.0/24 (User/Attacker Zone - Behind Router0)
- PC0: '192.168.1.10' (Normal user)
- PC1: '192.168.1.11' (Simulated attacker)
- Server0: '192.168.1.100' (Syslog)
- Switch0
- Router0

#### LAN 172.16.0.0/24 (Service Zone - Behind Router1)
- Server1: '203.0.113.1' (Web Server via NAT)
- Server2: Reserved
- Switch1
- Router1


#### Inter-router Link
- Router0 <=> Router1: '10.0.0.1/30' <=> '10.0.0.2/30'


###  Network Topology Diagram
![Network Topology](https://github.com/nedelcubianca/network-security-project/blob/index.html/Topology_img.png?raw=true)


##  2. IP Addressing & NAT Configuration
In this step, static IP addresses are manually assigned to all devices, and NAT is configured to allow public access to the internal web server.

###  IP Address Plan
| Device        | Interface          | IP Address   | Subnet Mask      | Default Gateway|
|---------------|--------------------|--------------|------------------|----------------|
| PC0           | NIC                | 192.168.1.10 | 255.255.255.0    | 192.168.1.1    |
| PC1           | NIC                | 192.168.1.11 | 255.255.255.0    | 192.168.1.1    |
| Server0       | NIC                | 192.168.1.100| 255.255.255.0    | 192.168.1.1    |
| Server1       | NIC                | 172.16.0.100 | 255.255.255.0    | 172.16.0.1     |
| Router0 (LAN) | GigabitEthernet0/0 | 192.168.1.1  | 255.255.255.0    | -              |
| Router0 (WAN) | GigabitEthernet0/1 | 10.0.0.1     | 255.255.255.252  | -              |
| Router1 (WAN) | GigabitEthernet0/1 | 10.0.0.2     | 255.255.255.252  | -              |
| Router1 (LAN) | GigabitEthernet0/0 | 172.16.0.1   | 255.255.255.0    | -              |
| NAT Address   | (Public IP)        | 203.0.113.1  | N/A              | -              |

###  Manual IP Configuration (PCs and Servers)
Example for PC0:
IP Address:       192.168.1.10
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.1.1
###  Inter-Router Link Configuration
#### On Router0:
interface Gig0/1
ip address 10.0.0.1 255.255.255.252
no shutdown
#### On Router1:
interface Gig0/1
ip address 10.0.0.2 255.255.255.252
no shutdown
### NAT Configuration on Router1
Objective: We access Server1 (172.16.0.100) from LAN 192.168.1.0/24 using the "public" address 203.0.113.1
#### On Router1:
conf t
interface Gig0/0
ip address 10.0.0.2 255.255.255.252 //-> the interface to Router0
ip nat outside
no shutdown

interface Gig0/1
ip address 172.16.0.1 255.255.255.0 //-> the interface to Server1 
ip nat inside
no shutdown
exit

ip nat inside source static 172.16.0.100 203.0.113.1

#### Optionally, on Router0:
We set a static route from Lan 192.168.1.0/24 to the public address 203.0.113.1 of the Server1 from Lan 172.16.0.0/24. 
ip route 203.0.113.0 255.255.255.0 10.0.0.2
##### Accessing the server by name:
Configuring DNS on Server0: we use Services tab, then activate the DNS, add a new registration with the name 'web1' and the address '203.0.113.1'

Accesing 'http://web1' from PC0:
![Demonstration of accesing web1 on PC0](https://github.com/nedelcubianca/network-security-project/blob/index.html/nat_pc0.png?raw=true)

Also, we can successfully access 'http://203.0.113.1' and 'https://203.0.113.1'.
Note: If we intend to configure an additional public IP address to be reachable by the 192.168.1.0/24 internal network, a separate and properly configured server is required. A single server cannot be assigned multiple distinct NAT public addresses for the same internal network segment.
## 3. ACL Implementation – Restricting Web Access
Access Control Lists (ACLs) are used to restrict traffic from the attacker device (PC1) to the internal web server (Server1) using both HTTP (port 80) and HTTPS (port 443). All other traffic remains permitted.
### Goal
- Deny PC1 (`192.168.1.11`) from accessing `Server1 (203.0.113.1)` via ports 80 and 443
- Allow all other traffic
###  ACL Configuration on Router0
#### Create an extended ACL (with log)
Router0(config)# access-list 100 deny tcp 192.168.1.11 0.0.0.0 203.0.113.1 0.0.0.0 eq 80 log
Router0(config)# access-list 100 deny tcp host 192.168.1.11 host 203.0.113.1 eq 443
Router0(config)# access-list 100 permit ip any any
#### Apply the ACL on the LAN interface
Router0(config)# interface GigabitEthernet0/0
Router0(config-if)# ip access-group 100 in
Router0(config-if)# exit
#### Test with PC1:
![Test with PC1 ACL ](https://github.com/nedelcubianca/network-security-project/blob/index.html/acl_pc1.png?raw=true)
#### Verify ACL Matches
![Verify ACL ](https://github.com/nedelcubianca/network-security-project/blob/index.html/verify_acl.png?raw=true)
##  Section 4 – Red Team Simulation: FTP Attack
In this section, we simulate a real-world scenario where an attacker attempts to exploit an exposed FTP server using weak credentials and limited permissions. This exercise demonstrates the risks associated with insecure services in a networked environment.

### Objective

- Deploy and configure an FTP server on Server0
- Create two FTP users: one with full permissions ('admin') and one with restricted access ('guest')
- Simulate a legitimate connection from PC0
- Simulate an unauthorized access attempt from PC1
- Analyze potential risks and security implications
- 
  ###  FTP Server Setup – Server0
 **IP Address**: '192.168.1.100'
 **Service**: FTP
 **Status**: ON
 #### 👥 FTP Users:
 username1 : 'admin'  password1 : 'adminpass'  permissions1 : 'read, write, delete, rename, list'
 username2 : 'guest'  password2 : 'pass123'  permissions2 : 'list'
 ![FTP Config](https://github.com/nedelcubianca/network-security-project/blob/index.html/ftp_config.png?raw=true)
 #### Step 1: Legitimate Access from PC0
 Open Command Prompt on PC0 and run: ftp 192.168.1.100
 Enter:  
Username: admin
Password: adminpass
![Legitimate Access](https://github.com/nedelcubianca/network-security-project/blob/index.html/access_pc0.png?raw=true)
