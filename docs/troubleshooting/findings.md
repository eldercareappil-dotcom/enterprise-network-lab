# Troubleshooting Findings

Problems I found by testing failure scenarios and checking whether the configuration really did what I intended. Each entry: symptom, root cause, fix, verification.

---

## 1. NAT failures were invisible

**Symptom:** None. That was the problem. Internet connectivity always worked.

**Root cause:** The ISP router had static routes back to the internal networks (192.168.0.0/24, .1.0/24, .2.0/24). If PAT broke, packets would leave with a private source address, and the ISP would still route the replies back. A connectivity test couldn't tell working NAT from broken NAT. A real ISP drops private addresses.

**Fix:**
- Removed the six private static routes from ISP-R1.
- Added Loopback0 `198.51.100.1/32` on ISP-R1 as a simulated Internet host.

**Verification:** STAFF-PC01 → 198.51.100.1 successful; `show ip nat translations` on R1 shows inside local `192.168.0.102` translated to inside global `203.0.113.1`.

**Takeaway:** A test that can't fail proves nothing.

---

## 2. Internet blackhole when R1's WAN link fails

**Symptom:** With R1 G0/1 (WAN) down, clients lost Internet access although both routers were healthy and HSRP looked normal.

**Root cause:** HSRP only monitors the LAN segment where its hellos run. R1's LAN side was up, so it stayed active. Its static default route was removed together with G0/1, and OSPF carried no default route, so R1 had no path to the Internet.

**Fix:** `default-information originate` under OSPF on R1 and R2. Each router advertises its static default into OSPF; each prefers its own static route (AD 1) over the neighbour's OSPF default (AD 110), which stays in the LSDB as a backup. Without `always`, a router stops advertising once its own static default disappears.

**Verification:** R1 G0/1 shut down →
- R1: `O*E2 0.0.0.0/0 [110/1] via 10.0.12.2, GigabitEthernet0/2`
- STAFF-PC01 → 198.51.100.1 successful
- `show ip nat translations` on R2: inside global `203.0.113.5`. R2 performs the NAT because its transit interface G0/2 is configured as `ip nat inside`.
- After `no shutdown`, R1's static default returns and the OSPF default leaves the routing table.

**Alternatives:** HSRP interface tracking (R2 becomes the active gateway). In production, IP SLA tracking would also catch upstream failures where the link stays up.

---

## 3. `passive-interface` statements had no effect

**Symptom:** OSPF config listed the VLAN subinterfaces as passive, but the VLANs weren't advertised.

**Root cause:** `passive-interface` only affects interfaces that already participate in OSPF. The subinterfaces were never added to OSPF, so the intent (advertise the VLANs, but don't send hellos toward users) was only half implemented.

**Fix:** `ip ospf 1 area 0` on G0/0.10, .20 and .30 on both routers (still passive).

**Verification:**
- Normal state: no change in R1's routing table. Connected routes (AD 0) win, and R2's VLAN routes stay in the LSDB.
- R1 G0/0 shut down: `O 192.168.0.0/24`, `192.168.1.0/24`, `192.168.2.0/24 [110/2] via 10.0.12.2`.
- `tracert` from STAFF-PC01: first hop `192.168.0.12` (R2) during the failure, `192.168.0.11` (R1) after preemption.

**Why it matters:** If a router loses its LAN link, it can still reach the VLANs through its neighbour, so it stays manageable (e.g. SSH from the MANAGEMENT VLAN) during the failure.

---

## 4. Stateless ACL dropped return traffic

**Symptom:** The MANAGEMENT VLAN (IT) couldn't reach STAFF PCs. Pings and any TCP session from MANAGEMENT to STAFF failed.

**Root cause:** The STAFF ACL is applied inbound on the STAFF subinterface and denies STAFF → MANAGEMENT. ACLs are stateless: the replies from STAFF PCs matched that deny like any other traffic.

**Intended policy:** IT can start connections to anything; STAFF can't start connections to MANAGEMENT or the servers (except DHCP, DNS and the web server).

**Fix:** New ACL `STAFF-IN-V2` with two entries before the deny statements:
```
permit tcp 192.168.0.0 0.0.0.255 192.168.1.0 0.0.0.255 established
permit icmp 192.168.0.0 0.0.0.255 192.168.1.0 0.0.0.255 echo-reply
```
`established` matches TCP segments with ACK or RST set, i.e. replies within an existing session. A STAFF-initiated SYN doesn't match and is still denied.

Packet Tracer doesn't show ACL sequence numbers, so lines couldn't be inserted. I built the new ACL in full and swapped it with `ip access-group` (no unfiltered moment), then removed the old ACL. Applied identically on R1 and R2 so the policy doesn't change during an HSRP failover.

**Verification:** MANAGEMENT-PC02 → STAFF-PC01 successful; STAFF-PC01 → MANAGEMENT-PC02 blocked. Counters: `echo-reply` 4 matches, `deny ... 192.168.1.0` 4 matches.

**Limits:** `established` only inspects TCP flags. It doesn't track sessions and doesn't cover UDP. True stateful inspection needs a firewall.

---

## 5. Packet Tracer showed the wrong HSRP state

**Symptom:** With R1 G0/0 administratively down, `show standby brief` on R1 still showed all groups as Active.

**Investigation:**
- `show ip interface brief` on R1: G0/0 and all subinterfaces down/down
- R2 log: `Standby -> Active` for groups 10, 20, 30; R2 shows standby router "unknown"
- `tracert` from STAFF-PC01: first hop 192.168.0.12 (R2)

**Conclusion:** R2 was the real active router. R1's output was stale (real IOS shows Init for an HSRP group on a down interface). This is a simulator limitation.

**Takeaway:** When outputs disagree, cross-check: interface state, peer view, logs and the actual traffic path.
