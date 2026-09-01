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

---

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


## WAN Routing Configuration

Before configuring the VPN, basic routing between both sites had to be validated.

The WAN path consists of three routers:

- `CDG-ROUTER`
- `ISP Router`
- `BRANCH-ROUTER`

The two enterprise edge routers use default routes toward the simulated ISP, while the ISP router contains static routes toward the two pfSense WAN networks.

### CDG Router

The CDG edge router connects the main-site pfSense firewall to the ISP router.

```bash
enable
configure terminal

interface GigabitEthernet0/0
 ip address 10.0.1.1 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 ip address 203.0.113.1 255.255.255.252
 no shutdown

ip route 0.0.0.0 0.0.0.0 203.0.113.2

end
write memory
```


<img width="698" height="523" alt="image" src="https://github.com/user-attachments/assets/e0a87e76-63a9-4777-ab1f-d9ca95d714a2" />

The default route sends all unknown traffic toward the simulated ISP.

### ISP Router

The ISP router interconnects both enterprise edge routers.

```bash
enable
configure terminal

interface GigabitEthernet0/0
 ip address 203.0.113.2 255.255.255.252
 no shutdown

interface GigabitEthernet0/1
 ip address 198.51.100.1 255.255.255.252
 no shutdown

ip route 10.0.1.0 255.255.255.252 203.0.113.1
ip route 10.0.2.0 255.255.255.252 198.51.100.2

end
write memory
```
<img width="802" height="457" alt="image" src="https://github.com/user-attachments/assets/850fe640-07bd-447f-a17a-251adabd6652" />

The ISP router therefore knows how to reach both pfSense-facing WAN networks.

### Branch Router

The branch router connects the remote-site pfSense firewall to the ISP router.

```bash
enable
configure terminal

interface GigabitEthernet0/1
 ip address 198.51.100.2 255.255.255.252
 no shutdown

interface GigabitEthernet0/0
 ip address 10.0.2.1 255.255.255.252
 no shutdown

ip route 0.0.0.0 0.0.0.0 198.51.100.1

end
write memory
```

<img width="780" height="486" alt="image" src="https://github.com/user-attachments/assets/b728cfe8-e3e9-4fc9-b42c-98f20fc6343b" />

## pfSense Configuration

Two pfSense firewalls were deployed, one at each site.

Each pfSense instance uses two interfaces:

- `WAN` toward the edge router
- `LAN` toward the internal network

### Main Site pfSense

```text
WAN
IP Address : 10.0.1.2/30
Gateway    : 10.0.1.1

LAN
IP Address : 192.168.1.1/24
```

The LAN interface acts as the default gateway for hosts located inside the main-site network.

### Branch Site pfSense

```
WAN
IP Address : 10.0.2.2/30
Gateway    : 10.0.2.1

LAN
IP Address : 172.16.1.1/24
```

The branch LAN hosts use `172.16.1.1` as their default gateway.

Before configuring IPsec, connectivity between both pfSense WAN interfaces was validated across the simulated Internet.

This step was important to confirm that the underlying routing infrastructure was functional before introducing the VPN layer.


---

## End-Host Configuration

Windows 10 virtual machines were used as end hosts inside both LANs.

The main site contains multiple Windows 10 systems connected through access switches, while another Windows 10 system is connected to the branch LAN.

The systems used for the final VPN validation were configured as follows.

### Main Site Test Host

```text
IP Address : 192.168.1.102/24
Gateway    : 192.168.1.1
```

### Branch Site Test Host
```
IP Address : 172.16.1.2/24
Gateway    : 172.16.1.1
```

The Windows 10 hosts were also used to access the pfSense web administration interfaces through their respective LAN gateways.


<img width="797" height="467" alt="image" src="https://github.com/user-attachments/assets/32e0c3d8-ff79-4876-af96-08321b02e0a2" />

<img width="795" height="459" alt="image" src="https://github.com/user-attachments/assets/83228865-8214-4337-91cb-dcbfac50f747" />

---

## IPsec Site-to-Site VPN

Once basic WAN routing was confirmed to be functional, an IPsec Site-to-Site VPN was configured between the two pfSense firewalls.

The purpose of the tunnel is to securely transport traffic between the two private LANs:

- Main Site LAN: `192.168.1.0/24`
- Branch LAN: `172.16.1.0/24`

The VPN configuration uses:

- **IKEv2**
- **IPsec**
- **Pre-Shared Key authentication**
- **AES-256 encryption**
- **SHA-256 integrity**
- **Diffie-Hellman Group 14**

The VPN peers are the WAN interfaces of both pfSense firewalls:

- Main Site pfSense WAN: `10.0.1.2`
- Branch Site pfSense WAN: `10.0.2.2`

The next step consists of configuring IPsec Phase 1 and Phase 2 on both firewalls.

## IPsec Phase 1

Phase 1 is responsible for establishing the IKE Security Association between the two VPN gateways.

Both pfSense firewalls must use matching cryptographic and authentication parameters in order for the tunnel negotiation to succeed.

### Main Site pfSense

<img width="1028" height="366" alt="image" src="https://github.com/user-attachments/assets/14bff21a-4c26-4ee6-ad96-16ae8d57e3fc" />

<img width="675" height="619" alt="image" src="https://github.com/user-attachments/assets/f2e1d645-2e82-49e0-b65d-5d1baba121d6" />

<img width="889" height="529" alt="image" src="https://github.com/user-attachments/assets/8ca574bf-f5f5-4357-91fa-fb909e41e80f" />

Save and apply.

### Branch Site pfSense

<img width="1027" height="683" alt="image" src="https://github.com/user-attachments/assets/845f5e18-8266-4778-9b7f-6d5393f0a4bf" />

<img width="889" height="529" alt="image" src="https://github.com/user-attachments/assets/1a36d61f-d750-4eab-9232-74345af46b8f" />

Save and apply.

---

The same Pre-Shared Key must be configured on both sides.

The cryptographic parameters used for Phase 1 are:
```
Encryption Algorithm : AES-256
Hash Algorithm       : SHA-256
Diffie-Hellman Group : 14
```
These parameters provide confidentiality, integrity, and secure key exchange during the IKE negotiation process.












