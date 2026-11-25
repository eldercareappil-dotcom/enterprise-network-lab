# Network Testing & Verification

This document records practical tests performed on the Frankfurt network to verify connectivity, redundancy, routing, service availability and selected security mechanisms.

Only tests that have been performed are marked as verified.

## 1. EtherChannel Failure Test

**Objective:** Verify that the EtherChannel remains operational after losing one physical member link.

**Procedure:**
- Disconnect one physical cable between SW1 and SW2.
- Observe the Port-channel and remaining member link.
- Test network connectivity while the link is disconnected.

**Expected result:** The remaining physical link maintains connectivity between the switches.

**Result:** Successful. Network connectivity remained operational after one EtherChannel member was disconnected.

**Conclusion:** LACP provided link redundancy during the tested single-member failure.

## 2. HSRP Failover Test

**Objective:** Verify that R2 takes over gateway responsibilities when R1 becomes unavailable on the LAN side.

**Procedure:**
- Disable R1's GigabitEthernet0/0 interface.
- Verify that R2 takes over the active HSRP role.
- Test network services during the failover.

**Services tested:**

| Service | Test | Result |
|---|---|---|
| DNS | Name resolution | Successful |
| HTTP | Web access | Successful |
| HTTPS | Secure web access | Successful |
| DHCP | Dynamic IP assignment | Successful |
| ICMP | Network reachability | Successful |

**Result:** Successful. All tested services remained operational after R1's LAN interface was disabled.

**Conclusion:** HSRP failover allowed R2 to take over gateway responsibilities while maintaining the tested network services.

## 3. OSPF Verification

**Objective:** Verify dynamic routing between R1 and R2 over the dedicated point-to-point transit link.

**Verification performed:**
- OSPF neighbour adjacency reached FULL.
- R1 learned R2's Loopback0 route (`2.2.2.2/32`).
- R2 learned R1's Loopback0 route (`1.1.1.1/32`).
- Reachability between the router loopbacks was confirmed.

**Result:** Successful.

## 4. Native VLAN Verification

**Objective:** Verify consistent native VLAN configuration across all trunk connections.

**Verification performed:**
- SW1 Fa0/4 and Port-channel1 use native VLAN 999.
- SW2 Fa0/4 and Port-channel1 use native VLAN 999.
- R1 and R2 have native 802.1Q subinterfaces for VLAN 999.
- VLANs 10, 20 and 30 are allowed on the switch trunks.

**Result:** Successful. Both switches and routers use the same native VLAN configuration.

## 5. Port Security Verification

**Objective:** Observe Port Security operation on client and server access ports.

**Verification performed:**
- Port Security status and secure MAC addresses were inspected.
- A replacement device was connected to a STAFF access port after disconnecting the original client.
- The replacement device was able to join the network.

**Result:** Port Security status and secure MAC learning were inspected. The replacement device was successfully relearned and was able to communicate on the network.

**Conclusion:** The test demonstrated the limitations of dynamic MAC learning as an endpoint access control mechanism. Port Security alone does not authenticate the identity of a connected device.

## 6. DHCP Snooping Verification

**Objective:** Verify DHCP functionality after implementing DHCP Snooping.

**Verification performed:**
- DHCP Snooping binding tables were inspected.
- DHCP leases were successfully renewed on all four client PCs.
- DHCP functionality was confirmed after implementation.

**Result:** Successful. All four clients successfully renewed their DHCP leases.

**Observed Packet Tracer limitation:** Trust could not be configured directly on the logical Port-channel. Trust was configured on the physical EtherChannel member interfaces. The logical Port-channel continued to appear untrusted in the DHCP Snooping status output despite successful DHCP testing.

## 7. Dynamic ARP Inspection (DAI) Verification

**Objective:** Verify connectivity across VLANs 10 and 20 after implementing DAI.

**Verification performed:**
- Interface trust states were inspected.
- ARP tables were cleared on the test PCs.
- STAFF connectivity was tested after fresh ARP resolution.
- MANAGEMENT connectivity was tested after fresh ARP resolution.
- DAI inspection statistics were inspected.

**Result:** Successful. STAFF and MANAGEMENT connectivity remained operational after fresh ARP resolution.

**Scope limitation:** DAI is enabled only on VLANs 10 and 20. VLAN 30 is excluded due to Packet Tracer limitations affecting static ARP authorization.

## 8. Rogue Device Test

**Objective:** Observe the behavior of the existing access-layer security mechanisms when a replacement device is connected to a client access port.

**Test performed:**
- Disconnect the original client from a STAFF access port.
- Connect a replacement device.
- Observe the resulting network connectivity.

**Result:** The replacement device was able to participate in the STAFF VLAN and communicate over the network.

**Conclusion:** The tested security mechanisms did not authenticate the replacement device. A device able to obtain a legitimate DHCP binding could participate in the network, demonstrating the limitations of these mechanisms as endpoint authentication controls.

## 9. Testing Scope

These tests cover selected redundancy, routing, connectivity and security scenarios. They do not represent exhaustive production acceptance testing.

Further security, failure-recovery and troubleshooting scenarios will be added as the project develops.
