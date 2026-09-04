# Service Tests

Test results for infrastructure services deployed on the Faculty of Theft network — corresponds to **Chapter 5.2** of the full report.

Screenshots referenced below are stored in [`../../docss/images/services/`](../../docss/images/services/).

---

## 5.2.1 Web Service

**Test:** Access the faculty website through the Web Server's IP address.

| Item | Value |
|---|---|
| Target | `http://192.168.40.201` |
| Method | Browser navigation from a client machine |
| Result | ✅ Pass — "Faculty of Theft System" homepage rendered correctly |

> Note: site was served over plain HTTP (browser shows "Not Secure") — no TLS certificate configured, which is expected for an internal lab service.

Evidence: [web-service-test.png](../../docss/images/services/web-service-test.png)

---

## 5.2.2 TFTP Service

**Test:** Upload/serve a large file from the TFTP server and validate transfer integrity and throughput.

**1) Prepare a 100MB test file on the TFTP server and grant read permission to clients**

```bash
sudo fallocate -l 100M /srv/tftp/test100MB.bin
sudo chmod 644 /srv/tftp/test*.bin
```

**2) Connect to the TFTP server and download in verbose mode**

```
$ tftp 192.168.40.202
tftp> verbose
Verbose mode on.
tftp> trace
Packet tracing on.
tftp> get test100MB.bin
getting from 192.168.40.202:test100MB.bin to test100MB.bin [netascii]
sent RRQ <file=test100MB.bin, mode=netascii>
received DATA <block=1, 512 bytes>
sent ACK <block=1>
received DATA <block=2, 512 bytes>
sent ACK <block=2>
...
```

**Result:** ✅ Pass — file transferred completely via RRQ/DATA/ACK exchange with no dropped blocks.

**3) Verify bandwidth usage during the download with Wireshark**

Captured the transfer's I/O graph in Wireshark to confirm sustained throughput during the download (peaks around 60–80 Mb/s at 1-second intervals).

Evidence: [tftp-transfer.png](../../docs/images/services/tftp-transfer.png), [tftp-wireshark-io-graph.png](../../docs/images/services/tftp-wireshark-io-graph.png)

---

## 5.2.3 DHCP Server

**Test:** Connect client machines to each VLAN's switch and confirm they receive a dynamic IP lease.

Method: `cat /etc/netplan/50-cloud-init.yaml` confirmed `dhcp4: true` on each client, then `ip a` confirmed the assigned address.

| # | VLAN | Assigned IP | Result |
|---|------|--------------|--------|
| 1 | VLAN 10 | 192.168.10.21/24 | ✅ Pass |
| 2 | VLAN 20 | 192.168.20.50/24 | ✅ Pass |
| 3 | VLAN 30 | 192.168.30.49/24 | ✅ Pass |

All three clients received an address within their respective VLAN's DHCP scope, confirming per-VLAN DHCP is correctly configured on the firewall.

---

## 5.2.4 DNS Server

**Test:** After confirming DHCP-assigned DNS server settings, resolve the internal domain name and reach the Web Server by hostname instead of IP.

| Item | Value |
|---|---|
| Domain | `fot.isas.itkmitl.lab` |
| Resolves to | 192.168.40.201 (Web Server) |

| # | VLAN | Packets | Loss | RTT min/avg/max/mdev (ms) | Result |
|---|------|---------|------|----------------------------|--------|
| 1 | VLAN 10 | 4/4 | 0% | 29.749 / 37.462 / 48.810 / 6.988 | ✅ Pass |
| 2 | VLAN 20 | 4/4 | 0% | 24.706 / 27.650 / 31.647 / 2.691 | ✅ Pass |
| 3 | VLAN 30 | 4/4 | 0% | 19.450 / 20.043 / 20.744 / 5.634 | ✅ Pass |

All three VLANs successfully resolved and reached the internal domain name, confirming DNS is functioning correctly network-wide.

---

## 5.2.5 Zabbix Monitoring Service

**Test:** Confirm Zabbix is actively collecting and displaying metrics from monitored hosts.

| # | Check | Result |
|---|---|---|
| 1 | Zabbix-Server self-monitoring — network interface traffic (eth0) | ✅ Data visible (bits received/sent, error/discard counters) |
| 2 | Zabbix-Server self-monitoring — system performance (Linux system load, CPU usage) | ✅ Data visible |
| 3 | Web-Server host — network traffic monitoring | ✅ Data visible |

All monitored hosts (Web Server, File Server, Zabbix Server itself) reported live metrics on their respective dashboards, confirming the monitoring pipeline (SNMP polling → Zabbix Server → Dashboards) is working end-to-end.

Evidence: [zabbix-network-interface.png](../../docs/images/services/zabbix-network-interface.png), [zabbix-system-performance.png](../../docs/images/services/zabbix-system-performance.png)

---

## 5.2.6 OPNsense Reporting Service

**Test:** Verify OPNsense's built-in dashboard and reporting tools accurately reflect system and traffic status.

| # | Check | Result |
|---|---|---|
| 1 | Lobby Dashboard — system info, gateway status, services, traffic graph | ✅ Pass — WAN_DHCP6 gateway shown active, all core services (System Configuration Daemon, Cron, Dnsmasq DNS/DHCP, Host discovery, Users and Groups) running |
| 2 | Reporting → Traffic — real-time in/out throughput per interface | ✅ Pass — Server, Vlan10gateway, Vlan20gateway, Vlan30gateway, and WAN interfaces all reporting live bps graphs |
| 3 | Services → Dnsmasq DNS & DHCP → Leases — IP/MAC pairing table | ✅ Pass — see table below |

**DHCP lease table (sample):**

| Interface | IP Address | MAC Address | Hostname |
|---|---|---|---|
| Vlan10gateway | 192.168.10.21 | 0c:0c:66:7c:00:00 | ubuntu |
| Vlan20gateway | 192.168.20.46 | 0c:86:2b:80:00:00 | — |
| Vlan30gateway | 192.168.30.25 | 0c:9d:28:c6:00:00 | ubuntu-VMware-Virtual-Platform |

Evidence: [opnsense-dashboard.png](../../docs/images/services/opnsense-dashboard.png), [opnsense-traffic-reporting.png](../../docs/images/services/opnsense-traffic-reporting.png), [opnsense-dhcp-leases.png](../../docs/images/services/opnsense-dhcp-leases.png)

---

## 5.2.7 SSH Service

**Test:** From the Admin-Desktop, SSH into every switch in the topology to confirm remote management access.

Command pattern used (older Cisco IOSv images require legacy algorithms to be explicitly re-enabled on modern SSH clients):

```bash
ssh -o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa cisco@<switch-ip>
```

| # | Target | IP Address | Result |
|---|--------|------------|--------|
| 1 | Switch-server | 192.168.40.254 | ✅ Pass |
| 2 | Switch-client | 192.168.10.254 | ✅ Pass |
| 3 | Switch-room1 | 192.168.10.253 | ✅ Pass |
| 4 | Switch-room2 | 192.168.20.252 | ✅ Pass |
| 5 | Switch-room3 | 192.168.30.251 | ✅ Pass |

All five switches accepted SSH login from the Admin-Desktop using the configured `cisco` credentials, confirming remote management access is available across every switch in the topology.

---

## Summary

| Service | Tests Run | Passed | Failed |
|---|---|---|---|
| Web Service | 1 | 1 | 0 |
| TFTP Service | 2 | 2 | 0 |
| DHCP Server | 3 | 3 | 0 |
| DNS Server | 3 | 3 | 0 |
| Zabbix Monitoring | 3 | 3 | 0 |
| OPNsense Reporting | 3 | 3 | 0 |
| SSH Service | 5 | 5 | 0 |
| **Total** | **20** | **20** | **0** |

All infrastructure services passed validation. See [`connectivity-tests.md`](connectivity-tests.md) for network-layer testing (VLAN connectivity, routing, internet access, CARP failover).
