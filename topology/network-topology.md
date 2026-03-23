# ISP Network Deployment — Topology Specification

## Overview

This lab deploys an ISP network in **Cisco Modeling Labs (CML)** using **CSR 1000v** (IOS-XE) for all devices requiring EVPN/VXLAN and **IOSv** for the upstream ISP router. IS-IS underlay with EVPN/VXLAN overlay throughout. Designed to fit within a 64 GB RAM CML instance (~41 GB total).

## Nodes Used in This Lab

### Core — Cisco CSR 1000v (IOS-XE)

| Hostname | Role        | Platform       | Tier |
|----------|-------------|----------------|------|
| core-01  | Core Router | CSR 1000v | Core |
| core-02  | Core Router | CSR 1000v | Core |

### Aggregation — Cisco CSR 1000v (IOS-XE)

| Hostname | Role        | Platform       | Tier        |
|----------|-------------|----------------|-------------|
| agg-01   | Aggregation | CSR 1000v | Aggregation |
| agg-02   | Aggregation | CSR 1000v | Aggregation |

### FISP (Fiber ISP) — PON Access — Cisco CSR 1000v (IOS-XE)

| Hostname | Role     | Platform        | Tier   |
|----------|----------|-----------------|--------|
| pon-01   | FISP PON | CSR 1000v  | Access |
| pon-02   | FISP PON | CSR 1000v  | Access |
| pon-03   | FISP PON | CSR 1000v  | Access |
| pon-04   | FISP PON | CSR 1000v  | Access |

### WISP (Wireless ISP) — TWR Access — Cisco CSR 1000v (IOS-XE)

| Hostname | Role       | Platform        | Tier   |
|----------|------------|-----------------|--------|
| twr-01   | WISP Tower | CSR 1000v  | Access |
| twr-02   | WISP Tower | CSR 1000v  | Access |
| twr-03   | WISP Tower | CSR 1000v  | Access |

### ISP — Cisco IOSv

| Hostname | Role       | Platform | Tier |
|----------|------------|----------|------|
| isp-01   | ISP Router | IOSv     | ISP  |

### Edge — Cisco CSR 1000v (IOS-XE)

| Hostname | Role        | Platform       | Tier |
|----------|-------------|----------------|------|
| edge-01  | Edge Router | CSR 1000v | Edge |
| edge-02  | Edge Router | CSR 1000v | Edge |

### Test — Cisco CSR 1000v (IOS-XE)

| Hostname   | Role       | Platform       | Tier |
|------------|------------|----------------|------|
| vxlan-test | VXLAN Test | CSR 1000v | Test |

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
  IOSv   CSR1000v       CSR1000v           CSR1000v            CSR1000v
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

### Loopback Addresses (100.127.x.x/32)

Addressing scheme based on [StubArea51 reference](https://stubarea51.net/2025/09/22/evpn-vxlan-interop-ipv4-ipv6-mikrotik-ip-infusion/).

| Device     | Interface  | IPv4 Address         | IPv6 Address                    |
|------------|------------|----------------------|---------------------------------|
| isp-01     | Loopback0  | 100.127.0.1/32       | 3fff:1ab:d127:d0::1/128         |
| edge-01    | Loopback0  | 100.127.0.11/32      | 3fff:1ab:d127:d0::11/128        |
| edge-02    | Loopback0  | 100.127.0.12/32      | 3fff:1ab:d127:d0::12/128        |
| core-01    | Loopback0  | 100.127.1.1/32       | 3fff:1ab:d127:d1::1/128         |
| core-02    | Loopback0  | 100.127.1.2/32       | 3fff:1ab:d127:d1::2/128         |
| agg-01     | Loopback0  | 100.127.1.21/32      | 3fff:1ab:d127:d1::21/128        |
| agg-02     | Loopback0  | 100.127.1.22/32      | 3fff:1ab:d127:d1::22/128        |
| twr-01     | Loopback0  | 100.127.50.101/32    | 3fff:1ab:d127:d50::101/128      |
| twr-02     | Loopback0  | 100.127.50.102/32    | 3fff:1ab:d127:d50::102/128      |
| twr-03     | Loopback0  | 100.127.50.103/32    | 3fff:1ab:d127:d50::103/128      |
| pon-01     | Loopback0  | 100.127.52.101/32    | 3fff:1ab:d127:d52::101/128      |
| pon-02     | Loopback0  | 100.127.52.102/32    | 3fff:1ab:d127:d52::102/128      |
| pon-03     | Loopback0  | 100.127.52.103/32    | 3fff:1ab:d127:d52::103/128      |
| pon-04     | Loopback0  | 100.127.52.104/32    | 3fff:1ab:d127:d52::104/128      |
| vxlan-test | Loopback0  | 100.127.51.1/32      | 3fff:1ab:d127:d51::1/128        |

> Loopbacks serve as VTEP source addresses for VXLAN tunnels. IPv6 addressing follows 3fff:1ab:d127 scheme from blog reference.

### Point-to-Point Links

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

> Interface names: GigabitEthernet for CSR 1000v and IOSv. `[VERIFY]` exact port assignments.

### Customer/Overlay Networks

| Network         | VNI  | VLAN | Domain | Description                |
|-----------------|------|------|--------|----------------------------|
| 198.18.104.0/24 | 1104 | 1104 | WISP   | IPv4 customer overlay      |
| 198.18.106.0/24 | 1106 | 1106 | WISP   | IPv6 customer overlay      |

> FISP (PON) overlay networks to be defined following the same VNI/VLAN pattern.

---

## Underlay Protocol — IS-IS

| Parameter      | Value                |
|---------------|----------------------|
| IS-IS Area     | 49.0051              |
| Instance Name  | sa51                 |
| IS Type        | Level-2 only         |
| Metric Style   | Wide                 |
| AFI Support    | IPv4 and IPv6 (dual-stack) |
| Scope          | All devices          |

### IS-IS System IDs (NET)

| Device     | System ID        | NET                          |
|------------|------------------|------------------------------|
| isp-01     | 1001.2700.0001   | 49.0051.1001.2700.0001.00    |
| edge-01    | 1001.2700.0011   | 49.0051.1001.2700.0011.00    |
| edge-02    | 1001.2700.0012   | 49.0051.1001.2700.0012.00    |
| core-01    | 1001.2700.1001   | 49.0051.1001.2700.1001.00    |
| core-02    | 1001.2700.1002   | 49.0051.1001.2700.1002.00    |
| agg-01     | 1001.2700.1021   | 49.0051.1001.2700.1021.00    |
| agg-02     | 1001.2700.1022   | 49.0051.1001.2700.1022.00    |
| twr-01     | 1001.2705.0101   | 49.0051.1001.2705.0101.00    |
| twr-02     | 1001.2705.0102   | 49.0051.1001.2705.0102.00    |
| twr-03     | 1001.2705.0103   | 49.0051.1001.2705.0103.00    |
| pon-01     | 1001.2705.2101   | 49.0051.1001.2705.2101.00    |
| pon-02     | 1001.2705.2102   | 49.0051.1001.2705.2102.00    |
| pon-03     | 1001.2705.2103   | 49.0051.1001.2705.2103.00    |
| pon-04     | 1001.2705.2104   | 49.0051.1001.2705.2104.00    |
| vxlan-test | 1001.2705.1001   | 49.0051.1001.2705.1001.00    |

> Single IS-IS area across the entire fabric. IS-IS provides the underlay routing for VXLAN VTEP reachability. System IDs follow the blog's convention: core/agg use 1001.2700.x, access uses 1001.2705.x.

---

## Overlay Protocol — BGP EVPN / VXLAN

### EVPN/VXLAN Domains

| Domain | Devices | Description |
|--------|---------|-------------|
| Core ↔ Edge | core-01, core-02, edge-01, edge-02 | EVPN/VXLAN between core and edge routers |
| agg-01 ↔ WISP | agg-01, twr-01, twr-02, twr-03 | EVPN/VXLAN from aggregation to wireless tower access |
| agg-02 ↔ FISP | agg-02, pon-01, pon-02, pon-03, pon-04 | EVPN/VXLAN from aggregation to fiber PON access |

### BGP EVPN Configuration

- **Address Family:** L2VPN EVPN, IPv4 Unicast, IPv6 Unicast
- **Encapsulation:** VXLAN (all EVPN domains)
- **VTEP Source:** Loopback interfaces (100.127.x.x/32)
- **Route Types:** Type-2 (MAC/IP), Type-3 (IMET), Type-5 (IP Prefix)
- **Timers:** Keepalive 5s, Hold 15s

### BGP Design

| Parameter        | Value                              |
|-----------------|------------------------------------|
| BGP AS          | 4208675309 (private)               |
| Route Reflector | core-01 (primary), core-02 (secondary) |
| RR Clients      | All VTEP devices                   |
| EVPN Peers      | Loopback-based iBGP                |

---

## Access Layer

### WISP — TWR Devices (twr-01 through twr-03)

- **Role:** Wireless Internet Service Provider — customer access via tower infrastructure
- **Platform:** Cisco CSR 1000v
- **Overlay:** EVPN/VXLAN (via agg-01)
- **Underlay:** IS-IS (area 49.0051, instance sa51)
- **Count:** 3 devices

### FISP — PON Devices (pon-01 through pon-04)

- **Role:** Fiber Internet Service Provider — customer access via PON
- **Platform:** Cisco CSR 1000v
- **Overlay:** EVPN/VXLAN (via agg-02)
- **Underlay:** IS-IS (area 49.0051, instance sa51)
- **Count:** 4 devices

### Interconnection with Core/Aggregation

- **agg-01 → twr-01, twr-02:** EVPN/VXLAN overlay with IS-IS underlay
- **agg-02 → pon-01, pon-02:** EVPN/VXLAN overlay with IS-IS underlay
- **IP addressing:** All transit link IPs assigned (see P2P links table)

---

## Device Access

| Parameter        | Value                          |
|-----------------|--------------------------------|
| SSH             | Enabled on all devices         |
| SSH Port        | 22                             |
| Username        | (see .env file)                |
| Password        | (see .env file)                |

> All devices configured with SSH access for remote management. Credentials stored in .env (gitignored).

### CDP

- **CDP enabled globally** on all devices at initial deployment
- Used for neighbor validation and topology verification

### External Access (isp-01)

- **isp-01** has an additional **external access interface** bridged to the CML host network
- Bridge interface IP is defined in `.env` as `ISP01_BRIDGE_IP`
- A **default route** on isp-01 points out this bridged interface, providing external reachability
- SSH from **isp-01** directly to every other device via loopback addresses
- isp-01 serves as the **SSH gateway** for the entire lab (no multi-hop required)

---

## VXLAN Configuration (CSR 1000v)

| Parameter        | Value                          |
|-----------------|--------------------------------|
| Flood Mode      | BGP-based (EVPN)              |
| MAC Learning    | Disabled (control-plane only) |
| NVE Interface   | nve1                          |

### VNI Definitions

| VNI  | VLAN | Type | Overlay Network    | Route Target   | Description       |
|------|------|------|--------------------|----------------|-------------------|
| 1104 | 1104 | L2   | 198.18.104.0/24    | 1104:1104      | IPv4 overlay      |
| 1106 | 1106 | L2   | 198.18.106.0/24    | 1106:1106      | IPv6 overlay      |

### Overlay Host Addresses (VNI 1104 — IPv4)

| Device | IP Address       |
|--------|------------------|
| twr-01 | 198.18.104.101/24 |
| twr-02 | 198.18.104.102/24 |
| twr-03 | 198.18.104.103/24 |

### Overlay Host Addresses (VNI 1106 — IPv6)

| Device | IP Address       |
|--------|------------------|
| twr-01 | 198.18.106.101/24 |
| twr-02 | 198.18.106.102/24 |
| twr-03 | 198.18.106.103/24 |

> VNI and overlay addresses from blog reference. PON/FISP overlay addresses to be defined.

---

## CML Deployment

| Parameter     | Value                              |
|--------------|------------------------------------|
| CML Instance | (see .env file)                    |
| API Endpoint | /api/v2                            |
| Lab Name     | isp-network-deployment             |
| Node Images  | CSR 1000v, IOSv               |
| Deployment   | Automated via CML REST API         |

### API Deployment Method

Lab will be deployed via the CML REST API using a YAML topology file.

- **Endpoint:** `POST /api/v2/import` — imports a complete lab from YAML
- **Python client:** `client.import_lab_from_path("topology.yaml")`

### CML YAML Topology Format

```yaml
lab:
  description: ''
  notes: ''
  title: lab-name
nodes:
  - id: n0
    label: device-hostname
    node_definition: csr1000v    # or iosv, unmanaged_switch
    configuration: |
      hostname device-hostname
      ...
    interfaces:
      - id: i0
        label: GigabitEthernet1
      - id: i1
        label: GigabitEthernet2
links:
  - id: l0
    n1: n0          # source node id
    n2: n1          # destination node id
    i1: i0          # source interface id
    i2: i0          # destination interface id
```

### Node Definitions

| Device Type      | CML node_definition | RAM    |
|-----------------|---------------------|--------|
| CSR 1000v       | csr1000v            | 3072 MB |
| IOSv            | iosv                | 512 MB  |
| Unmanaged Switch | unmanaged_switch   | —       |

### Staged Deployment Plan

CML server has 4 CPU cores — cannot run all nodes simultaneously. Lab is deployed in stages, testing each before moving on.

#### Stage 1 — Core Tier

| Device     | State |
|------------|-------|
| ext-conn   | ON    |
| isp-01     | ON    |
| edge-01    | ON    |
| edge-02    | ON    |
| core-01    | ON    |
| core-02    | ON    |
| agg-01     | ON    |
| agg-02     | ON    |
| twr-01–03  | OFF   |
| pon-01–04  | OFF   |
| pon-sw-01  | OFF   |
| vxlan-test | OFF   |

**Tests:** IS-IS adjacencies, BGP EVPN sessions, L3 EtherChannel, loopback reachability, SSH access.

#### Stage 2 — WISP (TWR)

| Device     | State |
|------------|-------|
| ext-conn   | ON    |
| isp-01     | ON    |
| edge-01    | ON    |
| edge-02    | ON    |
| core-01    | ON    |
| core-02    | ON    |
| agg-01     | ON    |
| agg-02     | OFF   |
| twr-01     | ON    |
| twr-02     | ON    |
| twr-03     | ON    |
| pon-01–04  | OFF   |
| pon-sw-01  | OFF   |
| vxlan-test | OFF   |

**Tests:** TWR IS-IS adjacencies, BGP EVPN to RRs, VXLAN NVE status, VNI 1104/1106 overlay connectivity.

#### Stage 3 — FISP (PON)

| Device     | State |
|------------|-------|
| ext-conn   | ON    |
| isp-01     | ON    |
| edge-01    | ON    |
| edge-02    | ON    |
| core-01    | ON    |
| core-02    | ON    |
| agg-01     | OFF   |
| agg-02     | ON    |
| twr-01–03  | OFF   |
| pon-01     | ON    |
| pon-02     | ON    |
| pon-03     | ON    |
| pon-04     | ON    |
| pon-sw-01  | ON    |
| vxlan-test | OFF   |

**Tests:** PON IS-IS adjacencies (including via pon-sw-01), BGP EVPN to RRs, loopback reachability to pon-04 (furthest device).

---

## Diagram Reference

See: [Network Image.png](./Network%20Image.png)

---

## Notes

- All values marked `[VERIFY]` need confirmation against the high-resolution topology diagram
- Interface names: Ethernet for Nexus 9000v, GigabitEthernet for IOSv
- ISP: isp-01 (IOSv — no EVPN/VXLAN needed, 512 MB RAM)
- All other devices: CSR 1000v (3 GB RAM each)
- Total RAM: ~41 GB (fits 64 GB CML instance with headroom)
- Customer cloud connections exist on both the left (agg) and right (access) edges
