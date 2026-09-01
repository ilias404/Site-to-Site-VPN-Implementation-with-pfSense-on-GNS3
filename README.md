# Site-to-Site VPN Implementation with pfSense on GNS3

## Project Overview

This project was carried out during a one-month internship at **CDG Maroc**.

The main objective was to design and implement a network lab capable of securely interconnecting a **main site** and a **remote branch** through a WAN environment simulating the Internet.

The proposed solution relies on:

- **GNS3** for network emulation
- **pfSense** as firewall and VPN gateway
- **IPsec** for the Site-to-Site VPN tunnel
- **IKEv2** for VPN negotiation
- Cisco routers to simulate enterprise edge routing and Internet connectivity
- Windows 10 virtual machines as end hosts for connectivity testing

The GNS3 topology is a simplified representation of an enterprise environment. Some technologies present in a real production network, such as **Cisco StackWise**, were intentionally not reproduced in the lab because they were not required for the VPN implementation.

---

## Objectives

The main objectives of this project were to:

- Design a network architecture representing a main site and a remote branch
- Simulate Internet connectivity between both sites
- Configure WAN routing between the different network devices
- Deploy pfSense firewalls at both locations
- Configure an IPsec Site-to-Site VPN
- Secure communication between the two private LANs
- Validate the tunnel using end-to-end connectivity tests
- Verify that traffic is successfully exchanged through the encrypted tunnel

---

## Network Architecture

The final GNS3 lab contains:

### Main Site

- Three access switches
- One core switch
- Multiple Windows 10 hosts
- One pfSense firewall
- One edge router

### Simulated Internet

- One ISP router connecting both sites

### Branch Site

- One access switch
- One Windows 10 host
- One pfSense firewall
- One edge router

The IPsec tunnel is established between the two pfSense firewalls, while the routers between both sites simulate an untrusted WAN environment.

<img width="1269" height="636" alt="image" src="https://github.com/user-attachments/assets/8b95d42c-a9e9-4547-98d9-86eb9af44f96" />

## IP Addressing Plan

The network was divided into separate LAN and WAN segments in order to clearly distinguish internal enterprise networks from point-to-point transit links.

The main site uses the private network `192.168.1.0/24`, while the branch site uses `172.16.1.0/24`.

Point-to-point links between routers and pfSense firewalls use `/30` subnets because only two usable IP addresses are required on each link.

| Segment | Device | IP Address |
|---|---|---|
| Main LAN | pfSense LAN | `192.168.1.1/24` |
| Main LAN | Main test workstation | `192.168.1.102/24` |
| Main pfSense ↔ CDG Router | pfSense WAN | `10.0.1.2/30` |
| Main pfSense ↔ CDG Router | CDG Router | `10.0.1.1/30` |
| CDG Router ↔ ISP | CDG Router | `203.0.113.1/30` |
| CDG Router ↔ ISP | ISP Router | `203.0.113.2/30` |
| ISP ↔ Branch Router | ISP Router | `198.51.100.1/30` |
| ISP ↔ Branch Router | Branch Router | `198.51.100.2/30` |
| Branch Router ↔ pfSense | Branch Router | `10.0.2.1/30` |
| Branch Router ↔ pfSense | pfSense WAN | `10.0.2.2/30` |
| Branch LAN | pfSense LAN | `172.16.1.1/24` |
| Branch LAN | Branch Windows 10 host | `172.16.1.2/24` |

The public-looking address ranges `203.0.113.0/24` and `198.51.100.0/24` were used only inside the laboratory environment to represent Internet-facing networks.

> **Screenshot suggestion:** If the addressing is already visible on the final GNS3 topology, the topology screenshot can be reused here instead of adding a second image.

```markdown
![IP addressing overview](screenshots/gns3-topology.png)












