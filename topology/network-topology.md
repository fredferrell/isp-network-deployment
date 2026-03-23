# ISP Network Deployment — Topology Specification

## Overview

This lab deploys a multi-vendor ISP network using **IP Infusion OcNOS** and **MikroTik RouterOS** devices. The IP Infusion side runs **EVPN/VXLAN** while the MikroTik side runs **EVPN/VLAN**. The two domains interconnect at the core layer.

## Nodes Used in This Lab

| Hostname | Role         | Platform             | Tier        |
|----------|-------------|----------------------|-------------|
| core-01  | Core Router  | IP Infusion OcNOS   | Core        |
| agg-01   | Aggregation  | IP Infusion OcNOS   | Aggregation |
| lwr-01   | Leaf/Access  | IP Infusion OcNOS   | Access      |
| lwr-02   | Leaf/Access  | IP Infusion OcNOS   | Access      |
| lwr-03   | Leaf/Access  | IP Infusion OcNOS   | Access      |

### MikroTik Side

| Hostname            | Role               | Platform          |
|--------------------|--------------------|-------------------|
| MikroTik Switches  | BGP/OSPF Topology  | MikroTik RouterOS |
| Legacy MCLAG Pair  | Legacy Redundancy  | MikroTik RouterOS |

> **Note:** Exact MikroTik hostnames are not clearly labeled in the topology diagram. `[VERIFY]` specific device names.

---

## Topology Structure

```
                        IP Infusion (EVPN/VXLAN)                    MikroTik (EVPN/VLAN)
                     ┌──────────────────────────────┐          ┌───────────────────────────┐
                     │                              │          │                           │
  ☁ Customer ──── lwr-01 ──┐                        │          │    ┌── MikroTik SW(s) ──┐ │
                     │      ├──── agg-01 ──── core-01 ─────────┤    │   (BGP Topology)   │ │
  ☁ Customer ──── lwr-02 ──┤                        │          │    │                     │ │
                     │      │                        │          │    │   (OSPF Topology)   │ │
  ☁ Customer ──── lwr-03 ──┘                        │          │    └─── Legacy MCLAG ────┘ │── ☁ Customer
                     │                              │          │         (Red Box)          │
                     └──────────────────────────────┘          └───────────────────────────┘
```

### Connection Summary

- **Customer clouds (3x)** connect to lwr-01, lwr-02, lwr-03 via bridge interfaces
- **lwr switches** uplink to **agg-01** via point-to-point links
- **agg-01** uplinks to **core-01** via point-to-point links (dual-homed) `[VERIFY]`
- **core-01** interconnects to the **MikroTik** domain
- **MikroTik switches** run both **BGP** and **OSPF** topologies
- **Legacy MCLAG pair** provides backward-compatible redundancy on the MikroTik side
- **Customer cloud (1x)** connects on the MikroTik far-right edge

---

## IP Addressing

### Loopback Addresses (100.64.0.x/32)

| Device   | Interface | Address          |
|----------|-----------|------------------|
| core-01  | lo        | 100.64.0.1/32 `[VERIFY]` |
| agg-01   | lo        | 100.64.0.2/32 `[VERIFY]` |
| lwr-01   | lo        | 100.64.0.3/32 `[VERIFY]` |
| lwr-02   | lo        | 100.64.0.4/32 `[VERIFY]` |
| lwr-03   | lo        | 100.64.0.5/32 `[VERIFY]` |

> Loopbacks also serve as VTEP source addresses for VXLAN tunnels.

### Point-to-Point Links (198.51.0.x/31)

| Link                  | Device A     | Interface | IP             | Device B     | Interface | IP             |
|-----------------------|-------------|-----------|----------------|-------------|-----------|----------------|
| core-01 ↔ agg-01 (1) | core-01     | FPP1      | 198.51.0.0/31 `[VERIFY]` | agg-01      | FPP1      | 198.51.0.1/31 `[VERIFY]` |
| core-01 ↔ agg-01 (2) | core-01     | FPP2      | 198.51.0.2/31 `[VERIFY]` | agg-01      | FPP2      | 198.51.0.3/31 `[VERIFY]` |
| agg-01 ↔ lwr-01 (1)  | agg-01      | FPP3      | 198.51.0.4/31 `[VERIFY]` | lwr-01      | FPP1      | 198.51.0.5/31 `[VERIFY]` |
| agg-01 ↔ lwr-01 (2)  | agg-01      | FPP4      | 198.51.0.6/31 `[VERIFY]` | lwr-01      | FPP2      | 198.51.0.7/31 `[VERIFY]` |
| agg-01 ↔ lwr-02 (1)  | agg-01      | FPP5      | 198.51.0.8/31 `[VERIFY]` | lwr-02      | FPP1      | 198.51.0.9/31 `[VERIFY]` |
| agg-01 ↔ lwr-02 (2)  | agg-01      | FPP6      | 198.51.0.10/31 `[VERIFY]`| lwr-02      | FPP2      | 198.51.0.11/31 `[VERIFY]`|
| agg-01 ↔ lwr-03 (1)  | agg-01      | FPP7      | 198.51.0.12/31 `[VERIFY]`| lwr-03      | FPP1      | 198.51.0.13/31 `[VERIFY]`|
| agg-01 ↔ lwr-03 (2)  | agg-01      | FPP8      | 198.51.0.14/31 `[VERIFY]`| lwr-03      | FPP2      | 198.51.0.15/31 `[VERIFY]`|

> Interface names (FPP numbers) are estimated from diagram labels. `[VERIFY]` exact port assignments.

### Customer/Bridge Networks (10.0.0.x)

| Device   | Interface  | Network           | Description       |
|----------|-----------|-------------------|-------------------|
| lwr-01   | BRG       | 10.0.0.0/24 `[VERIFY]`  | Customer network 1 |
| lwr-02   | BRG       | 10.0.0.0/24 `[VERIFY]`  | Customer network 2 |
| lwr-03   | BRG       | 10.0.0.0/24 `[VERIFY]`  | Customer network 3 |

> Bridge (BRG) interfaces visible on customer-facing links. Exact subnets and VLAN IDs need verification from the diagram.

---

## Underlay Protocol — IS-IS

| IS-IS Area   | Scope                               | Color in Diagram |
|-------------|--------------------------------------|-----------------|
| 44.0001     | Access tier (lwr switches)           | Green `[VERIFY]` |
| 50.0001     | Core/Aggregation tier                | Yellow `[VERIFY]`|

> IS-IS provides the underlay routing for VXLAN VTEP reachability across the IP Infusion fabric.

---

## Overlay Protocol — BGP EVPN

- **Address Family:** L2VPN EVPN
- **Encapsulation (IP Infusion):** VXLAN
- **Encapsulation (MikroTik):** VLAN
- **VTEP Source:** Loopback interfaces (100.64.0.x/32)
- **Route Types:** Type-2 (MAC/IP), Type-5 (IP Prefix) `[VERIFY]`

### BGP Design

| Parameter           | Value                     |
|--------------------|---------------------------|
| BGP AS             | `[VERIFY]` from diagram   |
| Route Reflector    | core-01 or agg-01 `[VERIFY]` |
| EVPN Peers         | Loopback-based iBGP `[VERIFY]` |

---

## MikroTik Side — EVPN/VLAN

### Protocols

- **BGP Topology:** BGP sessions between MikroTik switches for EVPN route exchange
- **OSPF Topology:** OSPF underlay routing between MikroTik switches
- **Legacy MCLAG:** Multi-Chassis LAG pair for backward-compatible redundancy (highlighted in red box in diagram)

### Interconnection with IP Infusion

- **Transit link:** core-01 (OcNOS) ↔ MikroTik edge switch
- **EVPN gateway:** Translates between VXLAN encapsulation (OcNOS) and VLAN encapsulation (MikroTik)
- **IP addressing:** `[VERIFY]` transit link IPs from diagram

---

## VXLAN Configuration

| Parameter        | Value                    |
|-----------------|--------------------------|
| VNI Range       | `[VERIFY]` from diagram  |
| VLAN-to-VNI Map | `[VERIFY]` from diagram  |
| Flood Mode      | BGP-based (EVPN)        |
| ARP Suppression | `[VERIFY]`              |

---

## Diagram Reference

See: [Network Image.png](./Network%20Image.png)

---

## Notes

- All values marked `[VERIFY]` need confirmation against the high-resolution topology diagram
- Interface names (FPP numbering) are estimated; verify against actual OcNOS port layout
- MikroTik device hostnames are not clearly visible in the provided image
- The Legacy MCLAG pair is highlighted in a red box on the far-right of the diagram
- Customer cloud connections exist on both the left (IP Infusion) and right (MikroTik) edges
