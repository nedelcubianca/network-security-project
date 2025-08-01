#  Network Security Simulation Project

This project simulates a secure network architecture using **Cisco Packet Tracer**, focusing on access control, NAT configuration, and Red Team attack simulation.

##  Objective

To design and implement a realistic network scenario where:
- Devices are assigned static IPs.
- **NAT** (Network Address Translation) is configured for public web access.
- **ACLs** (Access Control Lists) are applied to restrict unauthorized  HTTP/HTTPS access.
- **FTP Server** is used to simulate brute-force access attempts.
- **Red Team techniques** are simulated to test the resilience of the network.

##  Tools & Technologies

- Cisco Packet Tracer 
- Cisco IOS commands (CLI)
- GitHub Pages for documentation

##  Project Structure

1. Network Topology
2. IP Configuration & NAT Setup
3. ACL Implementation (HTTP/HTTPS restrictions)
4. Red Team Simulation – FTP Attack
5. Final Conclusions & Lessons Learned

##  1. Network Topology

This network simulates a segmented infrastructure with enforced access control and isolated services.

### Devices & Addresses

#### LAN 192.168.1.0/24 (User/Attacker Zone - Behind Router0)
- PC0: '192.168.1.10' (Normal user)
- PC1: '192.168.1.11' (Simulated attacker)
- Server0: '192.168.1.100' (DNS & FTP Server)
- Switch0
- Router0

#### LAN 172.16.0.0/24 (Service Zone - Behind Router1)
- Server1: '203.0.113.1' (Web Server via NAT)
- Switch1
- Router1

#### Inter-router Link
- Router0 <=> Router1: 10.0.0.1 <=> 10.0.0.2


###  Network Topology Diagram
![Network Topology](https://github.com/nedelcubianca/network-security-project/blob/index.html/Topology_img.png?raw=true)


##  2. IP Addressing & NAT Configuration
Static IP addresses were manually assigned, and NAT was configured to provide external access to the internal web server.

###  IP Address Plan
Device - Interface - IP Address - Subnet Mask - Default Gateway
PC0 - 192.168.1.10 - 255.255.255.0 - 192.168.1.1 
PC1 - 192.168.1.11 - 255.255.255.0 - 192.168.1.1
Server0 - 192.168.1.100 - 255.255.255.0 - 192.168.1.1
Server1 - 172.16.0.100 - 255.255.255.0 -  172.16.0.1    
Router0  - GigabitEthernet0/0 - 192.168.1.1 - 255.255.255.0  
Router0  - GigabitEthernet0/1 - 10.0.0.1 - 255.255.255.252 
Router1  - GigabitEthernet0/1 - 10.0.0.2 - 255.255.255.252  
Router1  - GigabitEthernet0/0 - 172.16.0.1  - 255.255.255.0 

###  Manual IP Configuration (PCs and Servers)
Example for PC0:
IP Address:       192.168.1.10
Subnet Mask:      255.255.255.0
Default Gateway:  192.168.1.1
![Example for PC0](https://github.com/nedelcubianca/network-security-project/blob/index.html/ip_config_pc0.png?raw=true)

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
Router1(config)# interface Gig0/1
Router1(config-if)# ip address 172.16.0.1 255.255.255.0
Router1(config-if)# ip nat inside
Router1(config-if)# no shutdown
Router1(config)# interface Gig0/0
Router1(config-if)# ip address 10.0.0.2 255.255.255.252 //-> the interface to Router0
Router1(config-if)# ip nat outside
Router1(config-if)# no shutdown
Router1(config)# ip nat inside source static 172.16.0.100 203.0.113.1 //-> the interface to Server1 

#### Static Routing, on Router0:
We set a static route from Lan 192.168.1.0/24 to the public address 203.0.113.1 of the Server1 from Lan 172.16.0.0/24. 
Router0(config)# ip route 203.0.113.0 255.255.255.0 10.0.0.2

##### DNS Setup:
- Server0 (192.168.1.100) is configured as a DNS server
- Registered name: web1 → 203.0.113.1
- PCs must set DNS server to 192.168.1.100
- 
![Accessing the server by name](https://github.com/nedelcubianca/network-security-project/blob/index.html/dns_pc0.png?raw=true)

![Accessing the server by name](https://github.com/nedelcubianca/network-security-project/blob/index.html/dns_pc1.png?raw=true)

Accesing 'http://web1' from PC0:
![Demonstration of accesing web1 on PC0](https://github.com/nedelcubianca/network-security-project/blob/index.html/nat_pc0.png?raw=true)

Also, we can successfully access 'http://203.0.113.1' and 'https://203.0.113.1'.

Note: If we intend to configure an additional public IP address to be reachable by the 192.168.1.0/24 internal network, a separate and properly configured server is required. A single server cannot be assigned multiple distinct NAT public addresses for the same internal network segment.

## 3. ACL Implementation – Restricting Web Access
Access Control Lists (ACLs) are used to restrict traffic from the attacker device (PC1) to the internal web server (Server1) using both HTTP (port 80) and HTTPS (port 443). All other traffic remains permitted.
### Goal
- Deny PC1 (192.168.1.11) from accessing Server1 (203.0.113.1) via ports 80 and 443
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
This scenario simulates an attacker attempting to connect to an exposed FTP server using weak credentials.

### Objective
- Deploy and configure an FTP server on Server0
- Create two FTP users: one with full permissions ('admin') and one with restricted access ('guest')
- Simulate a legitimate connection from PC0
- Simulate an unauthorized access attempt from PC1
- Analyze potential risks and security implications
 
  ###  FTP ServerConfiguration – Server0
 **IP Address**: '192.168.1.100'
 **Service**: FTP
 **Status**: ON
 #### 👥 FTP Users:
 username1 : 'admin'  password1 : 'adminpass'  permissions1 : 'read, write, delete, rename, list'
 username2 : 'guest'  password2 : 'pass123'  permissions2 : 'read'
 ![FTP Config](https://github.com/nedelcubianca/network-security-project/blob/index.html/ftp_config.png?raw=true)
 #### Step 1: Legitimate Access - PC0
Open Command Prompt on PC0 and run: ftp 192.168.1.100
Enter:  
Username: admin
Password: adminpass
Testing the permissions in the image below:

![Legitimate Access](https://github.com/nedelcubianca/network-security-project/blob/index.html/access_pc0.png?raw=true)

#### Step 2: Unauthorized Access – PC1 (Red Team Simulation)
Open Command Prompt on PC1 and run: ftp 192.168.1.100
Using weak credentials:
Username: guest
Password: pass123
![Access Attempt](https://github.com/nedelcubianca/network-security-project/blob/index.html/access_pc1.png?raw=true)

#### Security Analysis
- FTP transmits credentials in plaintext -> insecure
- Guest access, if not properly limited, could aid recon or exploitation
- Demonstrates the principle of **least privilege**

##  Section 5 – Final Conclusions & Lessons Learned
This project shows how insecure services, weak credentials, and poor segmentation can expose networks to attack, even in simulated environments.

 ###  Key Concepts Covered
 
| **Network Design**    | Segmented LANs with routers, switches, PCs, and servers                |
| **IP Addressing**     | Manual assignment of IPs across multiple subnets                       |
| **NAT Configuration** | Allowed internal devices to access external services via translation   |
| **Access Control**    | ACLs were applied to limit web access to certain devices               |
| **Red Team Testing**  | Simulated brute-force attack on an FTP server with limited permissions |

###  Lessons Learned
- **FTP is inherently insecure**  
  Using it without encryption or strong authentication poses major risks.

- **Weak credentials are dangerous**  
  Even limited guest access can be exploited during recon and lateral movement.

- **ACLs provide essential control**  
  Properly configured access control lists can block or allow specific traffic, enforcing policy boundaries.

- **Least Privilege Principle matters**  
  Granting users only the permissions they need reduces the impact of potential breaches.

- **Simulation tools are valuable**  
  Packet Tracer enables realistic testing of network behaviors and security controls in a controlled environment.

###  Cybersecurity Perspective

This project mimics real-world Red Team operations and provides a foundation for:
- Analyzing insecure protocols
- Identifying exposed services
- Demonstrating how attackers operate and how defenders can respond

### 📚 Future Improvements

- Replace FTP with **SFTP (SSH-based)** for secure file transfer
- Introduce **logging/monitoring mechanisms** ( simulated syslog with better reliability)
- Add **intrusion detection rules** using ACLs + behavior analysis
- Expand simulation to include VLANs, VPN, or port security

##  Project File Download

You can download and open the full network simulation in Cisco Packet Tracer:

 [Download .pkt file](./network-security-simulation.pkt)

> Make sure you have Cisco Packet Tracer

  ###  Author

**Nedelcu Bianca-Nicoleta**  
Systems Engineering Student  
Project: *Network Design & Red Team Simulation – Cisco Packet Tracer*

> "To secure the future, you have to keep hacking your limits."
