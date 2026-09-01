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














