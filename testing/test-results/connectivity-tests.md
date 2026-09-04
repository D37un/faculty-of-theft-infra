# Network Connectivity Tests

Test results for network-level connectivity across the Faculty of Theft infrastructure — corresponds to **Chapter 5.1** of the full report. All tests were run from client machines using `ping`, with source interfaces confirmed via `ip a` beforehand.

Screenshots referenced below are stored in [`../../docs/images/services/`](../../docs/images/services/).

---

## 5.1.1 Intra-VLAN Connectivity

Verifies that hosts within the same VLAN can reach each other.

| # | VLAN | Source IP | Destination IP | Packets | Loss | RTT min/avg/max/mdev (ms) | Result |
|---|------|-----------|-----------------|---------|------|----------------------------|--------|
| 1 | VLAN 10 | 192.168.10.30 | 192.168.10.12 | 4/4 | 0% | 5.815 / 12.121 / 26.460 / 8.337 | ✅ Pass |
| 2 | VLAN 20 | 192.168.20.46 | 192.168.20.14 | 4/4 | 0% | 8.261 / 12.680 / 25.001 / 7.120 | ✅ Pass |
| 3 | VLAN 30 | 192.168.30.20 | 192.168.30.16 | 4/4 | 0% | 5.094 / 10.993 / 19.407 / 5.237 | ✅ Pass |

<details>
<summary>Sample output — VLAN 10</summary>

```
$ ping -c 4 192.168.10.12
PING 192.168.10.12 (192.168.10.12) 56(84) bytes of data.
64 bytes from 192.168.10.12: icmp_seq=1 ttl=64 time=26.5 ms
64 bytes from 192.168.10.12: icmp_seq=2 ttl=64 time=8.59 ms
64 bytes from 192.168.10.12: icmp_seq=3 ttl=64 time=5.82 ms
64 bytes from 192.168.10.12: icmp_seq=4 ttl=64 time=7.63 ms

--- 192.168.10.12 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3004ms
rtt min/avg/max/mdev = 5.815/12.121/26.460/8.337 ms
```
</details>

---

## 5.1.2 Inter-VLAN Routing

Verifies that the firewall correctly routes traffic between VLANs through the configured firewall rules.

| # | Route | Source IP | Destination IP | Packets | Loss | RTT min/avg/max/mdev (ms) | Result |
|---|-------|-----------|-----------------|---------|------|----------------------------|--------|
| 1 | VLAN 10 → VLAN 20 | 192.168.10.30 | 192.168.20.46 | 4/4 | 0% | 27.901 / 32.252 / 39.641 / 4.436 | ✅ Pass |
| 2 | VLAN 10 → VLAN 30 | 192.168.10.30 | 192.168.30.16 | 4/4 | 0% | 28.397 / 48.755 / 94.317 / 26.505 | ✅ Pass |
| 3 | VLAN 20 → VLAN 30 | 192.168.20.46 | 192.168.30.16 | 4/4 | 0% | 28.670 / 30.308 / 32.055 / 1.278 | ✅ Pass |

> Note: VLAN 10 → VLAN 30 shows higher latency/variance than the other routes — likely additional hop or link load at capture time. Still within acceptable range and 0% packet loss.

---

## 5.1.3 Internet Connectivity

Verifies outbound NAT and DNS resolution from every VLAN, plus a live browser test.

**1) Ping to `1.1.1.1` from each VLAN**

| # | VLAN | Source Host | Packets | Loss | RTT min/avg/max/mdev (ms) | Result |
|---|------|--------------|---------|------|----------------------------|--------|
| 1 | VLAN 10 | PC4 | 37/37 | 0% | 46.714 / 64.353 / 119.201 / 14.913 | ✅ Pass |
| 2 | VLAN 20 | PC2 | 4/4 | 0% | 45.727 / 63.031 / 87.716 / 15.556 | ✅ Pass |
| 3 | VLAN 30 | PC6 | 14/14 | 0% | 50.364 / 68.118 / 143.254 / 23.967 | ✅ Pass |

**2) DNS lookup via browser**

Opened `google.com` in a client browser (VLAN client) — page loaded successfully, confirming both DNS resolution and internet reachability end-to-end.

---

## 5.1.4 First Hop Redundancy Protocol (CARP) Failover

Verifies that each VLAN's gateway responds via its CARP virtual IP, and that failover to the secondary firewall is seamless.

**1) Ping to each VLAN's gateway (virtual IP)**

| # | VLAN | Gateway Virtual IP | Result |
|---|------|---------------------|--------|
| 1 | VLAN 10 | 192.168.10.110 | ✅ Reachable |
| 2 | VLAN 20 | 192.168.20.120 | ✅ Reachable |
| 3 | VLAN 30 | 192.168.30.130 | ✅ Reachable |

**2) Failover test**

Procedure: while continuously pinging each gateway virtual IP, the link between **Switch-Client** and **Primary-RouterFirewall** was disconnected to force CARP failover to **Secondary-RouterFirewall**.

**Result:** ✅ Pass — ping continued with **no errors or timeouts** across all three VLANs during and after the failover, confirming CARP correctly promoted the secondary firewall without service interruption.

---

## Summary

| Test Category | Tests Run | Passed | Failed |
|---|---|---|---|
| Intra-VLAN Connectivity | 3 | 3 | 0 |
| Inter-VLAN Routing | 3 | 3 | 0 |
| Internet Connectivity | 4 | 4 | 0 |
| CARP Failover | 4 | 4 | 0 |
| **Total** | **14** | **14** | **0** |

All network-layer tests passed with 0% packet loss across the board. See [`service-tests.md`](service-tests.md) for service-level testing (Web, TFTP, DHCP, DNS, Zabbix, OPNsense Reporting, SSH).
