# NetArchitect

NetArchitect is an algorithmic network planning and self-explaining architecture engine built in Python 3.10+. It ingests enterprise network parameters from preset scenarios or custom topology JSON, validates them against network design constraints, and outputs copy-paste-ready Cisco IOS/IOS-XE configurations with explainable engineering rationale.

It is designed for network engineers, architects, and automation teams who want a fast path from topology design to deployable network configuration without sacrificing correctness, standards awareness, or documentation quality.

## Why NetArchitect?

- Dynamic bitwise VLSM subnet allocation with mathematically verified zero-overlap behavior
- Layer 1 media selection for Cat6a, OM4 MMF, and OS2 SMF
- Optical link-budget validation with loss and margin calculation
- Layer 3 routing decisions for OSPF and eBGP environments
- Cisco IOS configuration generation for VLAN trunking, subinterfaces, ACLs, and IPsec tunnels
- Explainable design logic based on standards and engineering rules
- Rich CLI and FastAPI-based dashboard for interactive usage

---

## System Architecture

```text
                    ┌────────────────────────────────────────┐
                    │            INPUT INGESTION             │
                    │ • Preset profile JSON                 │
                    │ • Custom topology JSON                │
                    └───────────────────┬────────────────────┘
                                        │
                     ┌────────────────────┴────────────────────┐
                     ▼                                         ▼
          ┌───────────────────────────┐          ┌───────────────────────────┐
          │     LAYER 1 MEDIA ENGINE   │          │      LAYER 3 VLSM ENGINE   │
          │ • Distance thresholds      │          │ • Descending sort         │
          │ • Cat6a vs OM4 vs OS2     │          │ • Power-of-2 sizing       │
          │ • Physical boundary check │          │ • Non-overlapping blocks  │
          │ • Optical budget analysis │          │ • Wildcard inversion      │
          └─────────────┬─────────────┘          └─────────────┬─────────────┘
                        │                                        │
                        └────────────────────┬────────────────────┘
                                             │
                                             ▼
                               ┌───────────────────────────┐
                               │      L3 ROUTING ENGINE     │
                               │ • Subnet count threshold  │
                               │ • Multi-site / multi-AS   │
                               │ • OSPF Area 0 vs multi   │
                               │ • eBGP peering & ASN     │
                               └────────────┬──────────────┘
                                             │
                                             ▼
                    ┌────────────────────────────────────────┐
                    │        EXPLAINABLE EXPORTER           │
                    │ • Cisco IOS config generation         │
                    │ • Router-on-a-stick VLAN subinterfaces │
                    │ • Policy ACL generation               │
                    │ • Architecture rationale report       │
                    └───────────────────┬────────────────────┘
                                        │
                     ┌────────────────────┴────────────────────┐
                     ▼                                         ▼
          ┌───────────────────────────┐          ┌───────────────────────────┐
          │         RICH CLI          │          │     FASTAPI DASHBOARD     │
          │ • Interactive terminal    │          │ • Modern dark UI         │
          │ • Colorized status tables│          │ • REST API              │
          │ • Direct .cfg export     │          │ • One-click download    │
          └───────────────────────────┘          └───────────────────────────┘
```

---

## Preset Scenarios

NetArchitect ships with three enterprise-scale scenarios:

### 1) Campus LAN (`campus_lan.json`)

- Scale: 3 buildings, link distance under 400 m
- Capacity: 1,600 endpoints across 10 subnets
- Root prefix: `10.10.0.0/16`
- Technologies: Cat6a copper, OM4 MMF, IEEE 802.1Q VLANs, Rapid-PVST+, single-area OSPFv2, router-on-a-stick
- Output: `exports/profile1_campus_lan_cisco.cfg`

### 2) Metro MAN (`metro_man.json`)

- Scale: FinTech HQ + 2 remote branch/data center sites
- Capacity: 450 users across 9 subnets
- Root prefix: `172.16.0.0/18`
- Technologies: OS2 single-mode fiber, optical link-budget analysis, single-area OSPF
- Output: `exports/profile2_metro_man_cisco.cfg`

### 3) Global WAN (`global_wan.json`)

- Scale: 4 global cities with cloud integration
- Capacity: 2,000+ endpoints across 12 subnets
- Root prefix: `192.168.0.0/16`
- Technologies: multi-area OSPF, eBGP, route summarization, route-based IPsec VPN tunnels
- Output: `exports/profile3_global_wan_cisco.cfg`

---

## Design Rules and Boundary Conditions

| Domain | Rule / Boundary Condition | Enforcement |
| --- | --- | --- |
| Layer 1 Media | Run <= 100 m | Selects Cat6a (10GBASE-T, IEEE 802.3an) |
| Layer 1 Media | Run 101 m - 400 m | Selects OM4 MMF (10GBASE-SR, IEEE 802.3ae) |
| Layer 1 Media | Run > 400 m | Selects OS2 SMF (10GBASE-LR/ER) |
| Layer 1 Boundary | Copper forced on link > 100 m | Raises `PhysicalDistanceViolationError` |
| Layer 1 Budget | Fiber optical power budget | Margin must remain > 0 dB |
| Layer 3 VLSM | Host requirement ordering | Sorted descending before allocation |
| Layer 3 VLSM | Address space capacity | Raises `SubnetExhaustionError` if required addresses exceed root block |
| Layer 3 VLSM | Subnet overlap prevention | Bitwise verification guarantees zero overlap |
| Layer 3 Routing | Single AS / < 50 subnets | Single-area OSPFv2 |
| Layer 3 Routing | Multi-site / > 50 subnets | Multi-area OSPFv2 |
| Layer 3 Routing | Multi-AS / cloud VPC | OSPF + eBGP redistribution |
| Cisco IOS | Switch trunking | Explicit VLAN pruning using `switchport trunk allowed vlan` |
| Cisco IOS | Router-on-a-stick | `encapsulation dot1Q` precedes `ip address` |
| Cisco IOS | Security policies | Extended ACLs generated using wildcard masks |

---

## Installation and Quick Start

### 1. Clone and install dependencies

```powershell
cd c:\Users\DELL\CN_Mini_Project
pip install -r requirements.txt
```

### 2. Run via the Rich CLI

```powershell
# List available preset scenarios
python -m netarchitect.cli list-presets

# Run a preset scenario
python -m netarchitect.cli run --preset campus_lan

# Run using a custom topology JSON file
python -m netarchitect.cli run --file path/to/custom_scenario.json

# Validate a scenario JSON file
python -m netarchitect.cli validate netarchitect/presets/campus_lan.json
```

### 3. Launch the interactive web dashboard

```powershell
python -m netarchitect.web
```

Then open:

```text
http://127.0.0.1:8000
```

The dashboard includes:

- one-click preset loader
- VLSM allocation breakdown tables
- physical media recommendation tables
- optical link-budget analysis
- copy-to-clipboard and direct `.cfg` download actions

---

## Automated Test Suite

The project includes a pytest suite for required design boundaries and routing logic.

```powershell
python -m pytest tests/ -v
```

### Coverage highlights

- `TestMediaEngine`
  - Cat6a within and at the 100 m boundary
  - OM4 MMF at 101 m and 400 m
  - OS2 SMF beyond 400 m and long-haul links
  - `PhysicalDistanceViolationError` for invalid copper/MMF assignments
  - optical margin and viability verification
- `TestVLSMEngine`
  - descending sort order validation
  - mathematically verified zero-overlap subnet allocation
  - subnet attribute validation
  - `SubnetExhaustionError` on overflow
- `TestWildcardMask`
  - wildcard calculations for common CIDR masks
  - complement invariants and bitwise correctness
- `TestRoutingEngine`
  - single-area vs multi-area OSPF decision switching
  - eBGP trigger conditions for multi-AS and cloud VPC designs
- `TestPresetIntegration`
  - end-to-end validation for Campus LAN, Metro MAN, and Global WAN
  - Cisco syntax verification
  - router subinterface generation
  - ACL and IPsec validation

---

## AI Defense and Code Auditing

NetArchitect follows strict AI safety and auditing expectations. Key post-mortems for networking defects are documented in [`ai_audit_log.md`](ai_audit_log.md).

Important issues tracked include:

1. Missing `encapsulation dot1Q` on router-on-a-stick interfaces
2. Incorrect media selection and forced-SMF runtime failure
3. Wildcard-mask confusion in ACL generation
4. Unicode/terminal encoding issues in Windows CLI output

---

## Repository Structure

```text
Mini-PROJECT/
├── ai_audit_log.md
├── README.md
├── requirements.txt
├── state.yaml
├── exports/
│   ├── profile1_campus_lan_cisco.cfg
│   ├── profile1_campus_lan_rationale.md
│   ├── profile2_metro_man_cisco.cfg
│   ├── profile2_metro_man_rationale.md
│   ├── profile3_global_wan_cisco.cfg
│   └── profile3_global_wan_rationale.md
├── netarchitect/
│   ├── __init__.py
│   ├── __main__.py
│   ├── cli.py
│   ├── exceptions.py
│   ├── models.py
│   ├── orchestrator.py
│   ├── web.py
│   ├── engines/
│   │   ├── __init__.py
│   │   ├── config_generator.py
│   │   ├── media_engine.py
│   │   ├── routing_engine.py
│   │   └── vlsm_engine.py
│   ├── presets/
│   │   ├── __init__.py
│   │   ├── campus_lan.json
│   │   ├── metro_man.json
│   │   └── global_wan.json
│   └── static/
│       ├── app.js
│       ├── index.html
│       └── style.css
└── tests/
    └── test_all.py
```

---

## Notes

This project is intended for simulation, design validation, and rapid network architecture prototyping rather than as a production networking appliance. It emphasizes correctness, engineering explainability, and standards-aware configuration generation.

## License

This repository snapshot does not include a license file. If you plan to distribute or reuse the project, add an explicit license such as MIT or Apache-2.0.

## Contribution

Contributions and issue reports are welcome. For feature requests or bug reports, open an issue in the repository and include the relevant preset or topology JSON used to reproduce the problem.
