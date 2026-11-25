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

### Port Security

**Decision:** Port Security is implemented on client-facing and server-facing access ports.

- Client-facing ports use dynamically learned secure MAC addresses.
- Client ports are configured with a maximum of one secure MAC address.
- Server-facing ports use sticky MAC learning.
- Ports use violation mode `shutdown`.
- Client ports use a five-minute absolute aging configuration in Packet Tracer.
- Server-facing sticky MAC entries do not use aging.

**Reasoning:** Port Security provides a basic Layer 2 restriction by limiting the MAC addresses permitted on individual switch ports. Dynamic MAC learning accommodates clients that may move between access ports, while sticky MAC learning is used for fixed server connections.

**Trade-offs and limitations:**

- Port Security restricts MAC addresses, not endpoint identity.
- MAC addresses can be spoofed.
- Dynamic secure MAC learning is not equivalent to endpoint authentication.
- The five-minute aging configuration observed in Packet Tracer uses absolute aging, not inactivity-based aging.
- Packet Tracer demonstrated that a dynamically learned secure MAC could be relearned after disconnecting the original device and connecting a replacement. This behavior should not be interpreted as authentication or authorization.
- Automatic errdisable recovery was not implemented because the required command was unsupported in the Packet Tracer switch CLI used in this lab.

### DHCP Snooping

**Decision:** DHCP Snooping is enabled globally and for VLANs 10, 20 and 30 on both switches.

DHCP Snooping trust is configured on interfaces that are expected to receive legitimate DHCP server or relay-related traffic. Client-facing interfaces remain untrusted.

**Reasoning:** DHCP Snooping helps prevent rogue DHCP server responses from entering the network through untrusted interfaces and builds a binding database containing dynamically assigned client information. This database can also be used by Dynamic ARP Inspection.

Option 82 insertion was retained in the Packet Tracer configuration. DHCP renewals were successfully tested on all four client PCs after implementation.

**Trade-offs and limitations:**

- Trust is assigned per interface and is not an authentication mechanism for the DHCP server itself.
- A trusted interface permits DHCP server messages entering through that interface, regardless of whether the physical device is the intended DHCP server.
- Packet Tracer did not support configuring DHCP Snooping trust directly on the logical Port-channel interface. Trust was configured on the physical EtherChannel member interfaces instead.
- Packet Tracer displayed the logical Port-channel as untrusted in the DHCP Snooping status output despite the physical members being trusted and DHCP functionality being verified. This behavior is documented as a simulator limitation or implementation-specific behavior and is not assumed to represent every real Catalyst platform.

### Dynamic ARP Inspection (DAI)

**Decision:** DAI is enabled on VLANs 10 and 20 on both switches. VLAN 30 is excluded.

DAI trust is configured on router-facing interfaces and physical EtherChannel member interfaces. Client-facing interfaces remain untrusted.

**Reasoning:** DAI validates ARP traffic on untrusted interfaces against DHCP Snooping bindings. VLANs 10 and 20 use DHCP and provide the required binding information.

Infrastructure-facing interfaces are trusted to permit legitimate ARP traffic from routers and devices beyond the local switch.

**VLAN 30 exclusion:**

VLAN 30 contains statically addressed servers without DHCP Snooping bindings. Static authorization through ARP ACLs or manual bindings was considered, but the Packet Tracer 2960 CLI used in this lab did not support the required configuration.

Trusting all server-facing interfaces would bypass DAI inspection on those ports, offering little meaningful protection.

**Decision:** VLAN 30 remains excluded from DAI due to Packet Tracer limitations and the lack of a reliable authorization method for the static server addresses.

## 3. Design Philosophy

The network prioritizes predictable operation, maintainability and practical redundancy over unnecessary complexity.

Design decisions should be based on the actual topology, traffic requirements and operational needs rather than implementing features simply because they are available.

Security features are implemented where they provide meaningful protection. Where simulator limitations prevent a reliable implementation, the limitation and resulting design decision are documented rather than implementing a feature without an effective validation mechanism.

## 4. Future Development

This document will evolve as the network grows. Planned additions and architectural changes will be documented when implemented and verified.
