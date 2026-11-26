# Frankfurt Network Lab

Packet Tracer project files documenting the progressive development of the Frankfurt enterprise network. Each file is a milestone; the name shows the previous step and what was added.

| # | File | Date | Added |
|---|---|---|---|
| 0 | 0NetworkLab | 18.11.2026 | Base topology: VLANs, router-on-a-stick |
| 1 | 1NetworkLab-SSH | 19.11.2026 | SSH-only device management |
| 2 | 2NetworkLab-SSH-Rapid-PVST+ | 19.11.2026 | Rapid-PVST+, SW1 as root bridge |
| 3 | 3NetworkLab-Rapid-PVST+-DNS | 19.11.2026 | DNS server |
| 4 | 4NetworkLab-DNS-ExtendedACL | 19.11.2026 | Extended ACL for the STAFF VLAN |
| 5 | 5NetworkLab-ExtendedACL-CentralizedDHCP | 20.11.2026 | Centralized DHCP with relay |
| 6 | 6NetworkLab-CentralizedDHCP-EtherChannel | 21.11.2026 | LACP EtherChannel between SW1 and SW2 |
| 7 | 7NetworkLab-EtherChannel-OSPF | 23.11.2026 | OSPF between R1 and R2 |
| 8 | 8NetworkLab-OSPF-PortSecurity-DHCPSnooping-DAI | 25.11.2026 | Layer 2 security: port security, DHCP snooping, DAI |
| 9 | 9NetworkLab-L2Security-FailoverFixes | 26.11.2026 | Fixes found by failure testing: WAN failover via OSPF default, VLANs in OSPF, realistic ISP + Internet host, ACL return traffic ([findings](../../docs/troubleshooting/findings.md)) |

`LabPasswords.txt` contains generated, lab-only credentials, published on purpose so others can explore the CLI. They aren't used anywhere else.
