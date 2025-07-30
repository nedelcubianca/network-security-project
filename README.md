# 🔐 Network Security Simulation Project

This project simulates a secure network architecture using **Cisco Packet Tracer**, focusing on access control, network monitoring, and offensive security testing.

## 🎯 Objective

To design and implement a realistic network scenario where:
- Devices are assigned static IPs.
- **NAT** (Network Address Translation) is configured.
- **ACLs** (Access Control Lists) are applied to restrict unauthorized access.
- **Syslog** is used to monitor and log security events.
- **Red Team techniques** are simulated to test the resilience of the network.

## 🛠 Tools & Technologies

- Cisco Packet Tracer 
- Cisco IOS commands (CLI)
- GitHub Pages for documentation

## 📁 Project Structure

1. Network Topology
2. IP Configuration & NAT Setup
3. ACL Implementation (HTTP/HTTPS restrictions)
4. Syslog Server Monitoring
5. Red Team Simulation (port scanning, bypass attempts)
6. Results & Security Improvements

---

## 🌐 1. Network Topology

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

---

###  Network Topology Diagram
![Network Topology](https://github.com/<nedelcubianca>/network-security-project/blob/main/topology.png?raw=true)
