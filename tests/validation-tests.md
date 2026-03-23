# ISP Network Deployment — Validation Tests

## Test Logging

All test results must be documented in `tests/logs/` with the following format:

- **Filename:** `validation-YYYY-MM-DD-HHMMSS.log`
- **Format:** Each test entry includes timestamp, device, command, result (PASS/FAIL), and output

```
[2026-03-23 14:05:12] DEVICE: core-01 | TEST: IS-IS adjacency | CMD: show isis neighbors
STATUS: PASS
OUTPUT:
  System Id       Type Interface     IP Address      State Holdtime Circuit Id
  core-02         L2   Gi0/0/0       100.126.1.42    UP    25       core-02.01
---
```

- Logs are gitignored to keep the repo clean
- A new log file is created for each validation run

---

## 1. External Access

- [ ] SSH to isp-01 via bridged external interface
- [ ] Verify default route on isp-01 points to bridged interface
- [ ] From isp-01, SSH to edge-01
- [ ] From isp-01, SSH to edge-02

## 2. Device Reachability — SSH from isp-01 to All Devices

- [ ] isp-01 → edge-01: `ssh cisco@100.127.0.11`
- [ ] isp-01 → edge-02: `ssh cisco@100.127.0.12`
- [ ] isp-01 → core-01: `ssh cisco@100.127.1.1`
- [ ] isp-01 → core-02: `ssh cisco@100.127.1.2`
- [ ] isp-01 → agg-01: `ssh cisco@100.127.1.21`
- [ ] isp-01 → agg-02: `ssh cisco@100.127.1.22`
- [ ] isp-01 → twr-01: `ssh cisco@100.127.50.101`
- [ ] isp-01 → twr-02: `ssh cisco@100.127.50.102`
- [ ] isp-01 → twr-03: `ssh cisco@100.127.50.103`
- [ ] isp-01 → pon-01: `ssh cisco@100.127.52.101`
- [ ] isp-01 → pon-02: `ssh cisco@100.127.52.102`
- [ ] isp-01 → pon-03: `ssh cisco@100.127.52.103`
- [ ] isp-01 → pon-04: `ssh cisco@100.127.52.104`
- [ ] isp-01 → vxlan-test: `ssh cisco@100.127.51.1`

## 3. IS-IS Underlay

### IS-IS Adjacency

- [ ] isp-01: `show isis neighbors` — verify adjacency with edge-01, edge-02
- [ ] edge-01: `show isis neighbors` — verify adjacency with isp-01, edge-02, core-01, core-02
- [ ] edge-02: `show isis neighbors` — verify adjacency with isp-01, edge-01, core-01, core-02
- [ ] core-01: `show isis neighbors` — verify adjacency with edge-01, edge-02, core-02, agg-01, agg-02
- [ ] core-02: `show isis neighbors` — verify adjacency with edge-01, edge-02, core-01, agg-01, agg-02
- [ ] agg-01: `show isis neighbors` — verify adjacency with core-01, core-02, twr-01, twr-02, vxlan-test
- [ ] agg-02: `show isis neighbors` — verify adjacency with core-01, core-02, pon-01, pon-02
- [ ] twr-01: `show isis neighbors` — verify adjacency with agg-01, twr-03
- [ ] twr-02: `show isis neighbors` — verify adjacency with agg-01, twr-03, pon-01
- [ ] twr-03: `show isis neighbors` — verify adjacency with twr-01, twr-02
- [ ] pon-01: `show isis neighbors` — verify adjacency with agg-02, twr-02, pon-03 (via pon-sw-01)
- [ ] pon-02: `show isis neighbors` — verify adjacency with agg-02, pon-03 (via pon-sw-01)
- [ ] pon-03: `show isis neighbors` — verify adjacency with pon-01, pon-02 (via pon-sw-01), pon-04
- [ ] pon-04: `show isis neighbors` — verify adjacency with pon-03

### IS-IS Configuration

- [ ] All devices: IS-IS instance name is `sa51`
- [ ] All devices: IS-IS area is `49.0051`
- [ ] All devices: IS type is `level-2-only`
- [ ] All devices: Metric style is `wide`
- [ ] Verify IS-IS database is consistent: `show isis database` on core-01

### IS-IS Loopback Reachability

- [ ] From core-01, ping all loopbacks:
  - [ ] core-02: 100.127.1.2
  - [ ] agg-01: 100.127.1.21
  - [ ] agg-02: 100.127.1.22
  - [ ] edge-01: 100.127.0.11
  - [ ] edge-02: 100.127.0.12
  - [ ] twr-01: 100.127.50.101
  - [ ] twr-02: 100.127.50.102
  - [ ] twr-03: 100.127.50.103
  - [ ] pon-01: 100.127.52.101
  - [ ] pon-02: 100.127.52.102
  - [ ] pon-03: 100.127.52.103
  - [ ] pon-04: 100.127.52.104
  - [ ] vxlan-test: 100.127.51.1
  - [ ] isp-01: 100.127.0.1

## 4. BGP EVPN

### BGP Sessions

- [ ] core-01: `show bgp l2vpn evpn summary` — all peers Established
- [ ] core-02: `show bgp l2vpn evpn summary` — all peers Established
- [ ] Verify BGP AS is 4208675309 on all devices
- [ ] Verify core-01 is route reflector: `show bgp l2vpn evpn neighbors` — check RR client status
- [ ] Verify core-02 is secondary route reflector

### BGP Peer Verification (per device)

- [ ] edge-01: BGP peer to core-01, core-02 — Established
- [ ] edge-02: BGP peer to core-01, core-02 — Established
- [ ] agg-01: BGP peer to core-01, core-02 — Established
- [ ] agg-02: BGP peer to core-01, core-02 — Established
- [ ] twr-01: BGP peer to core-01, core-02 — Established
- [ ] twr-02: BGP peer to core-01, core-02 — Established
- [ ] twr-03: BGP peer to core-01, core-02 — Established
- [ ] pon-01: BGP peer to core-01, core-02 — Established
- [ ] pon-02: BGP peer to core-01, core-02 — Established
- [ ] pon-03: BGP peer to core-01, core-02 — Established
- [ ] pon-04: BGP peer to core-01, core-02 — Established
- [ ] vxlan-test: BGP peer to core-01, core-02 — Established

### EVPN Route Types

- [ ] Verify Type-2 (MAC/IP) routes present: `show bgp l2vpn evpn route-type 2`
- [ ] Verify Type-3 (IMET) routes present: `show bgp l2vpn evpn route-type 3`
- [ ] Verify route targets 1104:1104 and 1106:1106 are imported correctly

## 5. VXLAN

### NVE Interface

- [ ] All Cat 8000v devices: `show nve interface nve1` — status is Up
- [ ] Verify VTEP source is Loopback0 on each device

### VNI Status

- [ ] Verify VNI 1104 is active: `show nve vni`
- [ ] Verify VNI 1106 is active: `show nve vni`
- [ ] Verify VLAN-to-VNI mapping: VLAN 1104 → VNI 1104, VLAN 1106 → VNI 1106

### VTEP Peer Discovery

- [ ] twr-01: `show nve peers` — verify peers twr-02, twr-03
- [ ] twr-02: `show nve peers` — verify peers twr-01, twr-03
- [ ] twr-03: `show nve peers` — verify peers twr-01, twr-02

## 6. VXLAN Overlay Connectivity

### VNI 1104 (IPv4 Overlay — 198.18.104.0/24)

- [ ] From twr-01 (198.18.104.101), ping twr-02 (198.18.104.102)
- [ ] From twr-01 (198.18.104.101), ping twr-03 (198.18.104.103)
- [ ] From twr-02 (198.18.104.102), ping twr-03 (198.18.104.103)

### VNI 1106 (IPv6 Overlay — 198.18.106.0/24)

- [ ] From twr-01 (198.18.106.101), ping twr-02 (198.18.106.102)
- [ ] From twr-01 (198.18.106.101), ping twr-03 (198.18.106.103)
- [ ] From twr-02 (198.18.106.102), ping twr-03 (198.18.106.103)

## 7. L3 EtherChannel (core-01 ↔ core-02)

- [ ] Verify port-channel is up: `show etherchannel summary`
- [ ] Verify L3 (routed) mode: `show interfaces port-channel`
- [ ] Ping core-02 (100.126.1.42) from core-01 (100.126.1.41) via port-channel
- [ ] Verify both member links are active

## 8. P2P Link Verification

### Edge ↔ ISP

- [ ] isp-01 → edge-01: ping 203.0.113.2
- [ ] isp-01 → edge-02: ping 203.0.113.10

### Edge ↔ Edge

- [ ] edge-01 → edge-02: ping 100.126.1.34

### Edge ↔ Core

- [ ] edge-01 → core-01: ping 100.126.1.2
- [ ] edge-01 → core-02: ping 100.126.1.10
- [ ] edge-02 → core-01: ping 100.126.1.18
- [ ] edge-02 → core-02: ping 100.126.1.26

### Core ↔ Aggregation

- [ ] core-01 → agg-01: ping 100.126.1.50
- [ ] core-01 → agg-02: ping 100.126.1.58
- [ ] core-02 → agg-01: ping 100.126.1.66
- [ ] core-02 → agg-02: ping 100.126.1.74

### Aggregation ↔ WISP

- [ ] agg-01 → twr-01: ping 100.126.50.2
- [ ] agg-01 → twr-02: ping 100.126.50.18
- [ ] twr-01 → twr-03: ping 100.126.50.10
- [ ] twr-02 → twr-03: ping 100.126.50.26

### Aggregation ↔ FISP

- [ ] agg-02 → pon-01: ping 100.126.52.2
- [ ] agg-02 → pon-02: ping 100.126.52.18
- [ ] pon-01 → pon-03 (via pon-sw-01): ping 100.126.52.11
- [ ] pon-02 → pon-03 (via pon-sw-01): ping 100.126.52.11
- [ ] pon-03 → pon-04: ping 100.126.52.34

### Cross-links

- [ ] twr-02 → pon-01: ping 100.126.255.2
- [ ] agg-01 → vxlan-test: ping 100.126.51.2

## 9. CDP Neighbor Validation

> CDP must be enabled globally on all devices during initial deployment.

### CDP Neighbor Count per Device

| Device     | Expected CDP Neighbors |
|------------|----------------------|
| isp-01     | edge-01, edge-02 |
| edge-01    | isp-01, edge-02, core-01, core-02 |
| edge-02    | isp-01, edge-01, core-01, core-02 |
| core-01    | edge-01, edge-02, core-02, agg-01, agg-02 |
| core-02    | edge-01, edge-02, core-01, agg-01, agg-02 |
| agg-01     | core-01, core-02, twr-01, twr-02, vxlan-test |
| agg-02     | core-01, core-02, pon-01, pon-02 |
| twr-01     | agg-01, twr-03 |
| twr-02     | agg-01, twr-03, pon-01 |
| twr-03     | twr-01, twr-02 |
| pon-01     | agg-02, twr-02, pon-03 (via pon-sw-01) |
| pon-02     | agg-02, pon-03 (via pon-sw-01) |
| pon-03     | pon-01, pon-02 (via pon-sw-01), pon-04 |
| pon-04     | pon-03 |
| vxlan-test | agg-01 |

### CDP Commands

- [ ] All devices: `show cdp neighbors` — verify expected neighbor count and hostnames
- [ ] All devices: `show cdp neighbors detail` — verify platform matches (CSR1000v or IOSv)
- [ ] Verify CDP is enabled globally: `show cdp`
- [ ] Verify CDP is enabled on all active interfaces: `show cdp interface`

## 10. End-to-End Ping Tests (Loopback to Loopback)

### From isp-01 (100.127.0.1) to all loopbacks

- [ ] → edge-01: ping 100.127.0.11
- [ ] → edge-02: ping 100.127.0.12
- [ ] → core-01: ping 100.127.1.1
- [ ] → core-02: ping 100.127.1.2
- [ ] → agg-01: ping 100.127.1.21
- [ ] → agg-02: ping 100.127.1.22
- [ ] → twr-01: ping 100.127.50.101
- [ ] → twr-02: ping 100.127.50.102
- [ ] → twr-03: ping 100.127.50.103
- [ ] → pon-01: ping 100.127.52.101
- [ ] → pon-02: ping 100.127.52.102
- [ ] → pon-03: ping 100.127.52.103
- [ ] → pon-04: ping 100.127.52.104
- [ ] → vxlan-test: ping 100.127.51.1

### From twr-03 (100.127.50.103) to FISP endpoints

- [ ] → pon-01: ping 100.127.52.101
- [ ] → pon-02: ping 100.127.52.102
- [ ] → pon-03: ping 100.127.52.103
- [ ] → pon-04: ping 100.127.52.104

### From pon-04 (100.127.52.104) to WISP endpoints

- [ ] → twr-01: ping 100.127.50.101
- [ ] → twr-02: ping 100.127.50.102
- [ ] → twr-03: ping 100.127.50.103

### From vxlan-test (100.127.51.1) across fabric

- [ ] → core-01: ping 100.127.1.1
- [ ] → core-02: ping 100.127.1.2
- [ ] → twr-01: ping 100.127.50.101
- [ ] → pon-01: ping 100.127.52.101
- [ ] → edge-01: ping 100.127.0.11

## 11. SSH Direct Access Verification

- [ ] From isp-01, SSH to the furthest WISP device (twr-03) directly — no hops
- [ ] From isp-01, SSH to the furthest FISP device (pon-04) directly — no hops
- [ ] From isp-01, SSH to vxlan-test directly — no hops

## 12. Redundancy / Failover

- [ ] Shut core-01 ↔ agg-01 link, verify traffic reroutes via core-02
- [ ] Shut edge-01 ↔ core-01 link, verify edge-01 reaches core via core-02
- [ ] Restore all links and verify convergence
