# ISP Network Deployment

A full ISP network lab built in **Cisco Modeling Labs (CML)**, featuring IS-IS underlay with BGP EVPN/VXLAN overlay. The topology models a dual-homed ISP with core, aggregation, and two distinct access domains: **WISP** (wireless tower) and **FISP** (fiber PON).

Designed to fit a 64 GB RAM CML instance (~41 GB total across 16 devices).

![Network Topology](topology/Network%20Image.png)

## Topology

```
  ISP      Edge           Core            Aggregation              Access

  isp-01 --+-- edge-01 --+-- core-01 --+-- agg-01 --+-- twr-01 --+-- twr-03
            |      |      |      ||     |            +-- twr-02 ---+
            +-- edge-02 --+-- core-02 --+              (WISP)
                                |
                                +-- agg-02 --+-- pon-01 --+
                                             |            +--[pon-sw-01]-- pon-03 -- pon-04
                                             +-- pon-02 --+
                                                 (FISP)
```

**16 devices** total: 2 edge, 2 core, 2 aggregation, 3 WISP towers, 4 FISP PON nodes, 1 ISP upstream, 1 VXLAN test node, 1 unmanaged switch.

## Key Design Decisions

| Layer | Technology | Details |
|-------|-----------|---------|
| Underlay | IS-IS | Area 49.0051, instance `sa51`, level-2 only, wide metrics |
| Overlay | BGP EVPN/VXLAN | AS 4208675309, core-01/02 as route reflectors |
| Core interconnect | L3 EtherChannel | Routed port-channel between core-01 and core-02 |
| Platforms | CSR 1000v (3 GB) / IOSv (512 MB) | IOSv only for isp-01 (no EVPN needed) |
| IP scheme | 100.127.x.x/32 loopbacks | Based on [StubArea51 EVPN reference](https://stubarea51.net/2025/09/22/evpn-vxlan-interop-ipv4-ipv6-mikrotik-ip-infusion/) |

## Staged Deployment

The CML server has 4 CPU cores, so the lab deploys in stages:

| Stage | What boots | What to test |
|-------|-----------|-------------|
| **1 - Core** | isp-01, edge-01/02, core-01/02, agg-01/02 | IS-IS, BGP EVPN, EtherChannel, loopback reachability |
| **2 - WISP** | Stage 1 + twr-01/02/03 (agg-02 off) | TWR IS-IS, EVPN to RRs, VXLAN VNI 1104/1106 |
| **3 - FISP** | Stage 1 + pon-01/02/03/04, pon-sw-01 (agg-01 off) | PON IS-IS, EVPN to RRs, pon-04 reachability |

## Getting Started

1. Copy the environment template and fill in your CML credentials:
   ```bash
   cp .env.example .env
   ```

2. Import the topology into CML:
   - Via API: `POST /api/v2/import` with `cml/topology.yaml`
   - Via UI: Upload `cml/topology.yaml` through the CML web interface

3. Start nodes following the [staged deployment plan](#staged-deployment)

4. Access devices via SSH through isp-01 (the SSH gateway):
   ```bash
   ssh <user>@<ISP01_BRIDGE_IP>          # into isp-01
   ssh <user>@100.127.1.1                # from isp-01 to core-01 (etc.)
   ```

## Validation

Validation tests are defined in [`tests/validation-tests.md`](tests/validation-tests.md) covering 12 categories:

1. External access & SSH gateway
2. Device reachability (all 14 managed devices)
3. IS-IS underlay (adjacency, config, loopback reachability)
4. BGP EVPN sessions
5. VXLAN NVE and VNI status
6. VXLAN overlay connectivity (VNI 1104/1106)
7. L3 EtherChannel
8. Point-to-point link verification
9. CDP neighbor validation
10. End-to-end loopback ping tests
11. SSH direct access to furthest devices
12. Redundancy / failover

Test results are logged to `tests/logs/` (gitignored).

## Project Structure

```
.
├── cml/
│   └── topology.yaml              # CML-exported lab topology
├── topology/
│   ├── network-topology.md         # Full topology specification
│   ├── isp-network-deployment.cml.yaml  # Original CML topology
│   └── Network Image.png          # Visual topology diagram
├── tests/
│   ├── validation-tests.md        # Validation test checklists
│   └── logs/                      # Test run logs (gitignored)
├── .env.example                   # Environment template
└── .gitignore
```

## Known Gotchas

- **CSR 1000v requires 3072 MB RAM minimum** - 2048 MB causes config load failure
- **EtherChannel on CSR 1000v requires an EEM applet** workaround to bring up the port-channel
- **4-core CML server constraint** means you cannot boot all 16 nodes simultaneously - follow the staged deployment plan

## References

- [StubArea51 EVPN/VXLAN Interop Guide](https://stubarea51.net/2025/09/22/evpn-vxlan-interop-ipv4-ipv6-mikrotik-ip-infusion/)
- [CML REST API Documentation](https://developer.cisco.com/docs/modeling-labs/)
