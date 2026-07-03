# Service Provider & Data Center Network Simulation with DevNet Automation

A production-style network simulation that combines a Service Provider core, a modern Leaf-Spine Data Center, Enterprise services, and network automation within a single integrated topology.

The objective of this project is to demonstrate routing design, traffic engineering, redundancy, enterprise security, and network automation using Cisco technologies.

##Network Architecture

The lab consists of four major components:

* Service Provider Core
* Broadband Network
* Leaf-Spine Data Center
* Enterprise Customer Network

The Service Provider provides Internet connectivity, IXP peering, and Broadband services while the Data Center hosts internal services connected through a Leaf-Spine architecture.


![Network Topology](diagrams/topology.png)

⸻
## Project Overview

This project demonstrates a full end-to-end network design covering:
- Service Provider core and edge
- Modern Leaf-Spine Data Center
- Enterprise customer network

⸻
## Key Technologies & Features

### Routing Design
OSPF Underlay Design

OSPF is deployed as the underlay routing protocol and is divided into multiple areas to simulate a scalable production network.

Area 0 – Backbone

Backbone-1 and Backbone-2 form the OSPF backbone (Area 0).

These routers interconnect every major section of the infrastructure, including:

* Data Center
* Broadband
* Edge Routers
* IXP

⸻

Area 1 – Edge

Edge-1 and Edge-2 are located in Area 1.

This area connects the Service Provider to multiple upstream Internet providers.

⸻

Area 2 – Broadband

Broadband-1, Broadband-2, Site-1 and POP-Site-1 belong to Area 2.

This area is configured as a Totally NSSA.

The objective is to inject only a default route toward customer-facing devices while preventing unnecessary external LSAs from entering the Broadband network.

Broadband-1 and Broadband-2 also operate as ABRs between Area 0 and Area 2.

⸻

Area 3 – Data Center

Spine-1, Spine-2 and Leaf-1, Leaf-2, Leaf-3 belong to Area 3.

This area is configured as a Totally Stub Area.

The Data Center only receives a default route from the Backbone.

As a result, Leaf and Spine switches only maintain:

* Local Data Center routes
* A single default route toward Backbone-1

This significantly reduces the routing table inside the Data Center.

Spine-1 and Spine-2 also operate as ABRs between Area 0 and Area 3.

⸻

Area 4 – IXP

The IXP Core is deployed inside Area 4 as a normal OSPF area.

⸻

BGP Overlay Design

BGP is deployed as the overlay routing protocol to simulate a production Service Provider network.

Instead of using a single Autonomous System, the provider is divided into multiple member Autonomous Systems using BGP Confederation.

Confederation Identifier:

64400

Member ASs:
```
Member AS   Components

65501 	Edge-1, Edge-2

65502 	Backbone-1, Backbone-2, IXP, Spine-1, Spine-2, Leaf-1, Leaf-2, Leaf-3

65503 	Broadband-1, Broadband-2, Site-1, POP-Site-1
```

The routing hierarchy is designed as:
```
AS65503

      │
      ▼
AS65502

      │
      ▼
AS65501
```
⸻

Core Routing Design

Backbone-1 acts as the primary aggregation router.

It connects directly to:

* Data Center
* Broadband
* Edge
* IXP

Since the Data Center is connected to Backbone-1, a default route is originated from Backbone-1 toward the Spine switches.

Each Spine router propagates that default route toward its connected Leaf switches.

Therefore, Leaf switches never receive Broadband or Internet routing tables.

Instead, they maintain only:

* Local Data Center routes
* One default route

This keeps the routing table compact while allowing full connectivity.

⸻

BGP Traffic Engineering

Several BGP attributes are implemented to simulate real-world traffic engineering.

Route Reflector

Backbone routers operate as Route Reflectors to eliminate the need for a full iBGP mesh and simplify route distribution across the Service Provider core.

⸻

IXP

The IXP peers with external networks using maximum-paths, allowing load balancing across multiple equal-cost paths.

⸻

Edge Policy

Edge routers are connected to multiple Internet providers.

The following BGP attributes are used:

* Weight
* Local Preference
* Origin
* AS-Path Prepending

These policies determine the preferred outbound Internet path while also influencing inbound traffic,outbound trafcic from upstream providers.

⸻

Broadband Policy

Inside AS65503, Broadband routers implement:

* Local Preference
* Weight
* AS-Path Prepending

Local Preference defines the preferred Broadband router for outbound traffic.

Weight and AS-Path Prepending influence send and return traffic from upstream networks.

⸻

Broadband Services

Broadband routers provide Internet access for customer networks.

Customer-facing POP routers assign private IP addressing.

Since CGNAT is not implemented in this lab, traditional NAT is configured on Broadband routers before traffic exits toward the Internet.

Traffic follows this path:

Customer

↓

POP

↓

Broadband

↓

Core

↓

IXP (if route exists)

↓

Otherwise Edge Routers

↓

Internet

⸻

Data Center

The Data Center follows a modern Leaf-Spine architecture.

Three test clients are connected for validation purposes.

DMVPN Phase 3 is deployed between these clients to demonstrate overlay VPN technologies.

These clients are independent from the Service Provider infrastructure and exist solely for testing DMVPN functionality.

Public routes are advertised toward the DMVPN network to validate end-to-end connectivity.
Traffic follows this path:

Customer

↓

leaf

↓

spine

↓

Core(broadband-1)

↓

IXP (if route exists)

↓

Otherwise Edge Routers

↓

Internet


⸻

Enterprise Network

An Enterprise campus is simulated behind the Broadband infrastructure.

The following Layer 2 and gateway technologies are implemented:

* HSRP
* EtherChannel
* Spanning Tree Protocol
* DHCP
* DHCP Snooping
* Dynamic ARP Inspection (DAI)

This section demonstrates common enterprise security and redundancy mechanisms.

### Automation (DevNet)
- **Python Backup System**:
  - DHCP IP assignment on management interface
  - Automated config backup using Python + Netmiko
- **EEM (Embedded Event Manager)**:
  - Monitors interface status on Core routers
  - Automatic reaction when a port goes down (BGP session management)

⸻
## Repository Structure
- `/diagrams` → Network topology
- `/configs` → Router configurations
- `/scripts` → Python automation scripts
- `/eem` → EEM applets
- `/docs` → documents(show bgp , show ip ospf database)
  
⸻
## Technologies

* Cisco IOS
* OSPF
* BGP Confederation
* Route Reflector
* BGP Traffic Engineering
* Leaf-Spine
* DMVPN Phase 3
* NAT
* IXP Peering
* HSRP
* EtherChannel
* Spanning Tree
* DHCP
* DHCP Snooping
* Dynamic ARP Inspection
* Python
* Netmiko
* Cisco EEM
* SSH






