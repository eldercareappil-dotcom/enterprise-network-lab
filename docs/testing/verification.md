# Network Testing & Verification

This document records practical tests performed on the Frankfurt network to verify connectivity, redundancy and service availability.

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

## 5. Testing Scope

These tests cover selected redundancy, routing and connectivity scenarios. They do not represent exhaustive production acceptance testing.

Further security, failure-recovery and troubleshooting scenarios will be added as the project develops.
