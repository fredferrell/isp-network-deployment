# ISP Network Deployment — Topology Specification

## Overview

This lab deploys an ISP network in **Cisco Modeling Labs (CML)** using **Nexus 9000v** for core/aggregation, **Catalyst 8000v** for edge/access/test devices requiring EVPN/VXLAN, and **IOSv** for the upstream ISP router. IS-IS underlay with EVPN/VXLAN overlay throughout.

## Nodes Used in This Lab

### Core — Cisco Nexus 9000v (NX-OS)

| Hostname | Role        | Platform      | Tier |
|----------|-------------|---------------|------|
| core-01  | Core Router | Nexus 9000v   | Core |
| core-02  | Core Router | Nexus 9000v   | Core |

### Aggregation — Cisco Nexus 9000v (NX-OS)

| Hostname | Role        | Platform      | Tier        |
|----------|-------------|---------------|-------------|
| agg-01   | Aggregation | Nexus 9000v   | Aggregation |
| agg-02   | Aggregation | Nexus 9000v   | Aggregation |

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

**Total: 15 devices**

---

## Topology Structure

```
  ISP      Edge           Core            Aggregation              Access
                                                 |
                                           +-- vxlan-test
                                           |
  isp-01 --+-- edge-01 --+-- core-01 --+-- agg-01 --+-- twr-01 --+-- twr-03
            |             |             |            +-- twr-02 ---+
            +-- edge-02 --+-- core-02 --+              (WISP)
                                        |
                                        +-- agg-02 --+-- pon-01 --+-- pon-03 -- pon-04
                                                     +-- pon-02 --+
                                                         (FISP)
  IOSv   Cat8000v     Nexus 9000v       Nexus 9000v           Cat8000v
```

### Connection Summary

- **isp-01** — Upstream ISP router, connects to **edge-01** and **edge-02**
- **edge-01, edge-02** connect to **core-01, core-02** via point-to-point links
- **core-01, core-02** connect to **agg-01, agg-02** via point-to-point links
- **vxlan-test** connects to **agg-01** (VXLAN testing)
- **agg-01** connects downstream to **twr-01, twr-02** — WISP topology (wireless, OSPF)
- **twr-03** connects to **twr-01** and **twr-02** (not directly to agg-01)
- **agg-02** connects downstream to **pon-01, pon-02** — FISP topology (fiber, BGP)
- **pon-03** connects to **pon-01** and **pon-02** (not directly to agg-02)
- **pon-04** connects to **pon-03** (not directly to agg-02)
- **Customer cloud (1x)** connects on the access far-right edge

---

## IP Addressing

### Loopback Addresses (100.64.0.x/32)

| Device   | Interface  | Address                   |
|----------|------------|---------------------------|
| core-01  | loopback0  | 100.64.0.1/32 `[VERIFY]`  |
| core-02  | loopback0  | 100.64.0.2/32 `[VERIFY]`  |
| agg-01   | loopback0  | 100.64.0.3/32 `[VERIFY]`  |
| agg-02   | loopback0  | 100.64.0.4/32 `[VERIFY]`  |
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

> Loopbacks serve as VTEP source addresses for VXLAN tunnels on Nexus 9000v devices.

### Point-to-Point Links (198.51.0.x/31)

| Link                  | Device A | Interface      | IP                        | Device B | Interface      | IP                        |
|-----------------------|----------|----------------|---------------------------|----------|----------------|---------------------------|
| isp-01 ↔ edge-01     | isp-01   | GigEth0/x          | `[VERIFY]`                | edge-01  | GigEth0/x          | `[VERIFY]`                |
| isp-01 ↔ edge-02     | isp-01   | GigEth0/x          | `[VERIFY]`                | edge-02  | GigEth0/x          | `[VERIFY]`                |
| edge-01 ↔ core-01    | edge-01  | GigEth0/x          | `[VERIFY]`                | core-01  | Ethernet1/x    | `[VERIFY]`                |
| edge-01 ↔ core-02    | edge-01  | GigEth0/x          | `[VERIFY]`                | core-02  | Ethernet1/x    | `[VERIFY]`                |
| edge-02 ↔ core-01    | edge-02  | GigEth0/x          | `[VERIFY]`                | core-01  | Ethernet1/x    | `[VERIFY]`                |
| edge-02 ↔ core-02    | edge-02  | GigEth0/x          | `[VERIFY]`                | core-02  | Ethernet1/x    | `[VERIFY]`                |
| core-01 ↔ agg-01     | core-01  | Ethernet1/x    | 198.51.0.0/31 `[VERIFY]`  | agg-01   | Ethernet1/x    | 198.51.0.1/31 `[VERIFY]`  |
| core-01 ↔ agg-02     | core-01  | Ethernet1/x    | 198.51.0.2/31 `[VERIFY]`  | agg-02   | Ethernet1/x    | 198.51.0.3/31 `[VERIFY]`  |
| core-02 ↔ agg-01     | core-02  | Ethernet1/x    | 198.51.0.4/31 `[VERIFY]`  | agg-01   | Ethernet1/x    | 198.51.0.5/31 `[VERIFY]`  |
| core-02 ↔ agg-02     | core-02  | Ethernet1/x    | 198.51.0.6/31 `[VERIFY]`  | agg-02   | Ethernet1/x    | 198.51.0.7/31 `[VERIFY]`  |
| agg-01 ↔ vxlan-test  | agg-01   | Ethernet1/x    | `[VERIFY]`                | vxlan-test | GigEth0/x        | `[VERIFY]`                |
| agg-01 ↔ twr-01      | agg-01   | Ethernet1/x    | `[VERIFY]`                | twr-01   | GigEth0/x          | `[VERIFY]`                |
| agg-01 ↔ twr-02      | agg-01   | Ethernet1/x    | `[VERIFY]`                | twr-02   | GigEth0/x          | `[VERIFY]`                |
| twr-01 ↔ twr-03      | twr-01   | GigEth0/x          | `[VERIFY]`                | twr-03   | GigEth0/x          | `[VERIFY]`                |
| twr-02 ↔ twr-03      | twr-02   | GigEth0/x          | `[VERIFY]`                | twr-03   | GigEth0/x          | `[VERIFY]`                |
| agg-02 ↔ pon-01      | agg-02   | Ethernet1/x    | `[VERIFY]`                | pon-01   | GigEth0/x          | `[VERIFY]`                |
| agg-02 ↔ pon-02      | agg-02   | Ethernet1/x    | `[VERIFY]`                | pon-02   | GigEth0/x          | `[VERIFY]`                |
| pon-01 ↔ pon-03      | pon-01   | GigEth0/x          | `[VERIFY]`                | pon-03   | GigEth0/x          | `[VERIFY]`                |
| pon-02 ↔ pon-03      | pon-02   | GigEth0/x          | `[VERIFY]`                | pon-03   | GigEth0/x          | `[VERIFY]`                |
| pon-03 ↔ pon-04      | pon-03   | GigEth0/x          | `[VERIFY]`                | pon-04   | GigEth0/x          | `[VERIFY]`                |

> Interface names: Ethernet for Nexus 9000v, GigabitEthernet for Catalyst 8000v/IOSv. `[VERIFY]` exact port assignments.

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

## VXLAN Configuration (Nexus 9000v)

| Parameter        | Value                   |
|-----------------|-------------------------|
| VNI Range       | `[VERIFY]` from diagram |
| VLAN-to-VNI Map | `[VERIFY]` from diagram |
| Flood Mode      | BGP-based (EVPN)       |
| ARP Suppression | `[VERIFY]`             |
| NVE Interface   | nve1                   |

---

## CML Deployment

| Parameter     | Value                          |
|--------------|--------------------------------|
| CML Instance | https://REDACTED         |
| Lab Name     | isp-network-deployment         |
| Node Images  | Nexus 9000v, Catalyst 8000v, IOSv |

---

## Diagram Reference

See: [Network Image.png](./Network%20Image.png)

---

## Notes

- All values marked `[VERIFY]` need confirmation against the high-resolution topology diagram
- Interface names: Ethernet for Nexus 9000v, GigabitEthernet for IOSv
- ISP: isp-01 (IOSv — no EVPN/VXLAN needed)
- Edge: edge-01, edge-02 (Catalyst 8000v)
- Core/Agg: core-01, core-02, agg-01, agg-02 (Nexus 9000v)
- Access: pon-01–04, twr-01–03 (Catalyst 8000v)
- Test: vxlan-test (Catalyst 8000v)
- Customer cloud connections exist on both the left (agg) and right (access) edges
