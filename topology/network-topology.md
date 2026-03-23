# ISP Network Deployment — Topology Specification

## Overview

This lab deploys an ISP network in **Cisco Modeling Labs (CML)** using **Catalyst 8000v** (IOS-XE) for all devices requiring EVPN/VXLAN and **IOSv** for the upstream ISP router. IS-IS underlay with EVPN/VXLAN overlay throughout. Designed to fit within a 64 GB RAM CML instance (~56.5 GB total).

## Nodes Used in This Lab

### Core — Cisco Catalyst 8000v (IOS-XE)

| Hostname | Role        | Platform       | Tier |
|----------|-------------|----------------|------|
| core-01  | Core Router | Catalyst 8000v | Core |
| core-02  | Core Router | Catalyst 8000v | Core |

### Aggregation — Cisco Catalyst 8000v (IOS-XE)

| Hostname | Role        | Platform       | Tier        |
|----------|-------------|----------------|-------------|
| agg-01   | Aggregation | Catalyst 8000v | Aggregation |
| agg-02   | Aggregation | Catalyst 8000v | Aggregation |

### FISP (Fiber ISP) — PON Access — Cisco Catalyst 8000v (IOS-XE)

| Hostname | Role     | Platform        | Tier   |
|----------|----------|-----------------|--------|
| pon-01   | FISP PON | Catalyst 8000v  | Access |
| pon-02   | FISP PON | Catalyst 8000v  | Access |
| pon-03   | FISP PON | Catalyst 8000v  | Access |
| pon-04   | FISP PON | Catalyst 8000v  | Access |

### WISP (Wireless ISP) — TWR Access — Cisco Catalyst 8000v (IOS-XE)

| Hostname | Role       | Platform        | Tier   |
|----------|------------|-----------------|--------|
| twr-01   | WISP Tower | Catalyst 8000v  | Access |
| twr-02   | WISP Tower | Catalyst 8000v  | Access |
| twr-03   | WISP Tower | Catalyst 8000v  | Access |

### ISP — Cisco IOSv

| Hostname | Role       | Platform | Tier |
|----------|------------|----------|------|
| isp-01   | ISP Router | IOSv     | ISP  |

### Edge — Cisco Catalyst 8000v (IOS-XE)

| Hostname | Role        | Platform       | Tier |
|----------|-------------|----------------|------|
| edge-01  | Edge Router | Catalyst 8000v | Edge |
| edge-02  | Edge Router | Catalyst 8000v | Edge |

### Test — Cisco Catalyst 8000v (IOS-XE)

| Hostname   | Role       | Platform       | Tier |
|------------|------------|----------------|------|
| vxlan-test | VXLAN Test | Catalyst 8000v | Test |

### Unmanaged Switch

| Hostname  | Role             | Platform              | Tier   |
|-----------|------------------|-----------------------|--------|
| pon-sw-01 | Unmanaged Switch | CML Unmanaged Switch  | Access |

**Total: 16 devices**

---

## Topology Structure

```
  ISP      Edge           Core            Aggregation              Access
                                                 |
                                           +-- vxlan-test
                                           |
  isp-01 --+-- edge-01 --+-- core-01 --+-- agg-01 --+-- twr-01 --+-- twr-03
            |      |      |      ||     |            +-- twr-02 ---+
            +-- edge-02 --+-- core-02 --+              (WISP) |
                   |         (L3 EtherChannel)                 |
              (edge link)              |              twr-02 --+-- pon-01
                                       |                (cross-link)  |
                                       +-- agg-02 --+-- pon-01 --+
                                                    |            +--[pon-sw-01]-- pon-03 -- pon-04
                                                    +-- pon-02 --+
                                                        (FISP)
  IOSv   Cat8000v       Cat8000v           Cat8000v            Cat8000v
```

### Connection Summary

- **isp-01** — Upstream ISP router, connects to **edge-01** and **edge-02**
- **edge-01 ↔ edge-02** interconnect via 100.126.1.32/29
- **edge-01, edge-02** connect to **core-01, core-02** via point-to-point links
- **core-01 ↔ core-02** interconnect via L3 EtherChannel (100.126.1.40/29, routed port-channel)
- **core-01, core-02** connect to **agg-01, agg-02** via point-to-point links
- **vxlan-test** connects to **agg-01** (VXLAN testing)
- **agg-01** connects downstream to **twr-01, twr-02** — WISP topology (wireless, OSPF)
- **twr-03** connects to **twr-01** and **twr-02** (not directly to agg-01)
- **twr-02 ↔ pon-01** cross-link between WISP and FISP (100.126.255.0/29)
- **agg-02** connects downstream to **pon-01, pon-02** — FISP topology (fiber, BGP)
- **pon-01, pon-02, pon-03** connect via **pon-sw-01** (unmanaged switch, 100.126.52.8/29)
- **pon-04** connects to **pon-03** (not directly to agg-02)
- **Customer cloud (1x)** connects on the access far-right edge

---

## IP Addressing

### Loopback Addresses (100.64.0.x/32)

| Device   | Interface  | Address                   |
|----------|------------|---------------------------|
| core-01  | Loopback0  | 100.64.0.1/32 `[VERIFY]`  |
| core-02  | Loopback0  | 100.64.0.2/32 `[VERIFY]`  |
| agg-01   | Loopback0  | 100.64.0.3/32 `[VERIFY]`  |
| agg-02   | Loopback0  | 100.64.0.4/32 `[VERIFY]`  |
| isp-01   | Loopback0  | `[VERIFY]`                |
| edge-01  | Loopback0  | `[VERIFY]`                |
| edge-02  | Loopback0  | `[VERIFY]`                |
| pon-01   | Loopback0  | `[VERIFY]`                |
| pon-02   | Loopback0  | `[VERIFY]`                |
| pon-03   | Loopback0  | `[VERIFY]`                |
| pon-04   | Loopback0  | `[VERIFY]`                |
| twr-01   | Loopback0  | `[VERIFY]`                |
| twr-02   | Loopback0  | `[VERIFY]`                |
| twr-03     | Loopback0  | `[VERIFY]`                |
| vxlan-test | Loopback0  | `[VERIFY]`                |

> Loopbacks serve as VTEP source addresses for VXLAN tunnels on Catalyst 8000v devices.

### Point-to-Point Links (198.51.0.x/31)

| Link                  | Device A | Interface      | IP                        | Device B | Interface      | IP                        |
|-----------------------|----------|----------------|---------------------------|----------|----------------|---------------------------|
| isp-01 ↔ edge-01     | isp-01   | GigEth0/x          | 203.0.113.1/29            | edge-01  | GigEth0/x          | 203.0.113.2/29            |
| isp-01 ↔ edge-02     | isp-01   | GigEth0/x          | 203.0.113.9/29            | edge-02  | GigEth0/x          | 203.0.113.10/29           |
| edge-01 ↔ edge-02    | edge-01  | GigEth0/x          | 100.126.1.33/29           | edge-02  | GigEth0/x          | 100.126.1.34/29           |
| edge-01 ↔ core-01    | edge-01  | GigEth0/x          | 100.126.1.1/29            | core-01  | GigEth0/x          | 100.126.1.2/29            |
| edge-01 ↔ core-02    | edge-01  | GigEth0/x          | 100.126.1.9/29            | core-02  | GigEth0/x          | 100.126.1.10/29           |
| edge-02 ↔ core-01    | edge-02  | GigEth0/x          | 100.126.1.17/29           | core-01  | GigEth0/x          | 100.126.1.18/29           |
| edge-02 ↔ core-02    | edge-02  | GigEth0/x          | 100.126.1.25/29           | core-02  | GigEth0/x          | 100.126.1.26/29           |
| core-01 ↔ core-02 (L3 EtherChannel) | core-01 | Port-channel (routed) | 100.126.1.41/29 | core-02 | Port-channel (routed) | 100.126.1.42/29 |
| core-01 ↔ agg-01     | core-01  | GigEth0/x          | 100.126.1.49/29           | agg-01   | GigEth0/x          | 100.126.1.50/29           |
| core-01 ↔ agg-02     | core-01  | GigEth0/x          | 100.126.1.57/29           | agg-02   | GigEth0/x          | 100.126.1.58/29           |
| core-02 ↔ agg-01     | core-02  | GigEth0/x          | 100.126.1.65/29           | agg-01   | GigEth0/x          | 100.126.1.66/29           |
| core-02 ↔ agg-02     | core-02  | GigEth0/x          | 100.126.1.73/29           | agg-02   | GigEth0/x          | 100.126.1.74/29           |
| agg-01 ↔ vxlan-test  | agg-01   | GigEth0/x          | 100.126.51.1/29           | vxlan-test | GigEth0/x          | 100.126.51.2/29           |
| agg-01 ↔ twr-01      | agg-01   | GigEth0/x          | 100.126.50.1/29           | twr-01   | GigEth0/x          | 100.126.50.2/29           |
| agg-01 ↔ twr-02      | agg-01   | GigEth0/x          | 100.126.50.17/29          | twr-02   | GigEth0/x          | 100.126.50.18/29          |
| twr-01 ↔ twr-03      | twr-01   | GigEth0/x          | 100.126.50.9/29           | twr-03   | GigEth0/x          | 100.126.50.10/29          |
| twr-02 ↔ twr-03      | twr-02   | GigEth0/x          | 100.126.50.25/29          | twr-03   | GigEth0/x          | 100.126.50.26/29          |
| twr-02 ↔ pon-01      | twr-02   | GigEth0/x          | 100.126.255.1/29          | pon-01   | GigEth0/x          | 100.126.255.2/29          |
| agg-02 ↔ pon-01      | agg-02   | GigEth0/x          | 100.126.52.1/29           | pon-01   | GigEth0/x          | 100.126.52.2/29           |
| agg-02 ↔ pon-02      | agg-02   | GigEth0/x          | 100.126.52.17/29          | pon-02   | GigEth0/x          | 100.126.52.18/29          |
| pon-01 ↔ pon-sw-01   | pon-01   | GigEth0/x          | 100.126.52.9/29           | pon-sw-01 | —                 | — (L2 unmanaged)          |
| pon-02 ↔ pon-sw-01   | pon-02   | GigEth0/x          | 100.126.52.10/29          | pon-sw-01 | —                 | — (L2 unmanaged)          |
| pon-sw-01 ↔ pon-03   | pon-sw-01 | —                 | — (L2 unmanaged)          | pon-03   | GigEth0/x          | 100.126.52.11/29          |
| pon-03 ↔ pon-04      | pon-03   | GigEth0/x          | 100.126.52.33/29          | pon-04   | GigEth0/x          | 100.126.52.34/29          |

> Interface names: GigabitEthernet for Catalyst 8000v and IOSv. `[VERIFY]` exact port assignments.

### Customer/Bridge Networks (10.0.0.x)

| Device   | Interface  | Network                 | Description      |
|----------|------------|-------------------------|------------------|
| agg-01   | `[VERIFY]` | 10.0.0.0/24 `[VERIFY]`  | Customer network |
| agg-02   | `[VERIFY]` | 10.0.0.0/24 `[VERIFY]`  | Customer network |

> Exact subnets and VLAN IDs need verification from the diagram.

---

## Underlay Protocol — IS-IS

| IS-IS Area | Scope         |
|-----------|---------------|
| 49.0051   | All devices   |

> Single IS-IS area across the entire fabric. IS-IS provides the underlay routing for VXLAN VTEP reachability.

---

## Overlay Protocol — BGP EVPN / VXLAN

### EVPN/VXLAN Domains

| Domain | Devices | Description |
|--------|---------|-------------|
| Core ↔ Edge | core-01, core-02, edge-01, edge-02 | EVPN/VXLAN between core and edge routers |
| agg-01 ↔ WISP | agg-01, twr-01, twr-02, twr-03 | EVPN/VXLAN from aggregation to wireless tower access |
| agg-02 ↔ FISP | agg-02, pon-01, pon-02, pon-03, pon-04 | EVPN/VXLAN from aggregation to fiber PON access |

### BGP EVPN Configuration

- **Address Family:** L2VPN EVPN
- **Encapsulation:** VXLAN (all EVPN domains)
- **VTEP Source:** Loopback interfaces (100.64.0.x/32)
- **Route Types:** Type-2 (MAC/IP), Type-5 (IP Prefix) `[VERIFY]`

### BGP Design

| Parameter        | Value                              |
|-----------------|------------------------------------|
| BGP AS          | `[VERIFY]` from diagram            |
| Route Reflector | core-01, core-02 `[VERIFY]`       |
| EVPN Peers      | Loopback-based iBGP `[VERIFY]`    |

---

## Access Layer

### WISP — TWR Devices (twr-01 through twr-03)

- **Role:** Wireless Internet Service Provider — customer access via tower infrastructure
- **Platform:** Cisco IOSv
- **Overlay:** EVPN/VXLAN (via agg-01)
- **Underlay:** IS-IS (area 49.0051)
- **Count:** 3 devices

### FISP — PON Devices (pon-01 through pon-04)

- **Role:** Fiber Internet Service Provider — customer access via PON
- **Platform:** Cisco IOSv
- **Overlay:** EVPN/VXLAN (via agg-02)
- **Underlay:** IS-IS (area 49.0051)
- **Count:** 4 devices

### Interconnection with Core/Aggregation

- **agg-01 → twr-01, twr-02:** EVPN/VXLAN overlay with IS-IS underlay
- **agg-02 → pon-01, pon-02:** EVPN/VXLAN overlay with IS-IS underlay
- **IP addressing:** `[VERIFY]` transit link IPs from diagram

---

## VXLAN Configuration (Catalyst 8000v)

| Parameter        | Value                   |
|-----------------|-------------------------|
| VNI Range       | `[VERIFY]` from diagram |
| VLAN-to-VNI Map | `[VERIFY]` from diagram |
| Flood Mode      | BGP-based (EVPN)       |
| ARP Suppression | `[VERIFY]`             |
| NVE Interface   | nve1 `[VERIFY]`        |

---

## CML Deployment

| Parameter     | Value                          |
|--------------|--------------------------------|
| CML Instance | https://REDACTED         |
| Lab Name     | isp-network-deployment         |
| Node Images  | Catalyst 8000v, IOSv              |

---

## Diagram Reference

See: [Network Image.png](./Network%20Image.png)

---

## Notes

- All values marked `[VERIFY]` need confirmation against the high-resolution topology diagram
- Interface names: Ethernet for Nexus 9000v, GigabitEthernet for IOSv
- ISP: isp-01 (IOSv — no EVPN/VXLAN needed, 512 MB RAM)
- All other devices: Catalyst 8000v (4 GB RAM each)
- Total RAM: ~56.5 GB (fits 64 GB CML instance)
- Customer cloud connections exist on both the left (agg) and right (access) edges
