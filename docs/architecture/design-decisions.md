# Frankfurt — Architecture & Design Decisions

## 1. Architecture

A small enterprise network built in Cisco Packet Tracer, consisting of:

- Two Cisco 2911 routers (R1 and R2)
- Two Cisco 2960 Layer 2 switches (SW1 and SW2)
- Three VLANs: STAFF, MANAGEMENT and SERVERS
- Three servers providing DNS, DHCP, FTP and Web services

The design focuses on network segmentation, redundancy, dynamic routing and practical network security.

## 2. Key Design Decisions & Trade-offs

### VLAN Distribution Across Switches

**Decision:** VLANs 10 (STAFF), 20 (MANAGEMENT) and 30 (SERVERS) are available across both switches.

**Reasoning:** Departments are not permanently tied to individual floors or switches. Office consolidation and changing occupancy can require employees from different departments to share the same physical office space.

For example, I have personally seen a six-floor office building reduced to three floors. The network should accommodate these changes without requiring departments to be moved between VLANs simply because their physical location changes.

**Trade-off:** Both switches must carry the required VLANs, making consistent VLAN and trunk configuration important.

### STP Root Bridge Placement

**Decision:** SW1 is the Root Bridge for VLANs 10, 20 and 30.

**Alternative considered:** Distribute the Root Bridge role between SW1 and SW2 to influence traffic paths.

**Reasoning:** STAFF and MANAGEMENT clients are distributed across both switches. Without traffic measurements demonstrating a significant imbalance, splitting the root roles offers no proven benefit. Keeping one root provides a predictable and simpler Layer 2 topology.

All servers are directly connected to SW1, which further supports keeping SW1 as the root for VLAN 30.

**Trade-off:** Traffic from devices connected to SW2 may need to cross the EtherChannel to reach resources connected through SW1.

### HSRP Active Gateway Placement

**Decision:** R1 is the active HSRP router for all three VLANs. R2 remains standby.

**Alternative considered:** Distribute active gateway roles between R1 and R2.

**Reasoning:** A single active gateway provides a consistent forwarding design and simplifies operations and troubleshooting. There is no measured traffic imbalance that justifies distributing gateway roles.

**Trade-off:** Traffic from clients connected to SW2 may cross the inter-switch EtherChannel to reach R1. This is accepted in favour of a simpler and predictable design.

### EtherChannel

**Decision:** LACP is used between SW1 and SW2.

**Reasoning:** Aggregating the physical links provides link redundancy and increased aggregate bandwidth while presenting a single logical connection to STP.

**Trade-off:** Both ends must have compatible EtherChannel settings. A configuration mismatch can prevent member links from joining the bundle.

### Routing

**Decision:** OSPF is used between R1 and R2 over a dedicated point-to-point transit link.

**Reasoning:** Dynamic routing allows the routers to exchange routing information rather than relying exclusively on manually configured routes.

### Native VLAN

**Decision:** VLAN 999 is used as the native VLAN on trunk connections instead of VLAN 1.

**Reasoning:** A dedicated, unused native VLAN separates untagged trunk traffic from the default VLAN and reduces the risk of relying on VLAN 1 for native traffic.

**Trade-off:** The native VLAN must match on both ends of every trunk, including router-on-a-stick connections.

## 3. Design Philosophy

The network prioritizes predictable operation, maintainability and practical redundancy over unnecessary complexity.

Design decisions should be based on the actual topology, traffic requirements and operational needs rather than implementing features simply because they are available.

## 4. Future Development

This document will evolve as the network grows. Planned additions and architectural changes will be documented when implemented and verified.
