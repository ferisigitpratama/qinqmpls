# QinQ over MPLS Lab with MikroTik and Cisco

A lab project that demonstrates **QinQ (802.1Q tunneling) over provider transport** using **MikroTik routers** and a **Cisco switch**. The lab is designed to simulate how multiple customer VLANs can be transported across a shared provider network while preserving VLAN separation for each site.

This project uses **RO-DIST** as the central distribution router, **RO-SITE-A / RO-SITE-B / RO-SITE-C** as remote customer sites, and **SW-QINQ** as the provider-edge switch performing **QinQ encapsulation** with **VLAN 500** as the service VLAN.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Objectives](#objectives)
- [Topology Overview](#topology-overview)
- [Architecture Summary](#architecture-summary)
- [VLAN and IP Plan](#vlan-and-ip-plan)
- [Device Roles](#device-roles)
- [How QinQ Works in This Lab](#how-qinq-works-in-this-lab)
- [Traffic Flow](#traffic-flow)
- [Configuration Summary](#configuration-summary)
- [Verification and Testing](#verification-and-testing)
- [Repository Contents](#repository-contents)
- [Suggested Screenshots](#suggested-screenshots)
- [Use Cases](#use-cases)
- [Conclusion](#conclusion)
- [Author](#author)

---

## Project Overview

In many enterprise and service provider environments, customer traffic from different VLANs needs to be transported across a shared backbone without changing the original customer-side VLAN design. One common way to achieve this is by using **QinQ**, also known as **802.1Q tunneling**, where the provider adds an **outer VLAN tag** to carry customer-tagged traffic across the transport network.

This lab simulates that scenario by combining **MikroTik routers** on the customer side and a **Cisco switch** on the provider edge. The design allows multiple site VLANs to be aggregated at a central router and transported through the provider network using a single service VLAN.

The project focuses on how **customer VLANs 10, 20, and 30** can be preserved and transported through the provider side using **VLAN 500** as the outer service tag.

---

## Objectives

This project was built to simulate a simple provider transport design that can:

- Carry multiple customer VLANs across a shared provider network
- Preserve customer VLAN separation end-to-end
- Demonstrate how QinQ encapsulation works in practice
- Show the relationship between **customer VLANs** and **provider service VLANs**
- Provide a hands-on lab for **MikroTik + Cisco interoperability**
- Validate site-to-distribution connectivity across QinQ transport

---

## Topology Overview

The lab consists of:

- **1 central distribution router**: `RO-DIST`
- **3 remote site routers**: `RO-SITE-A`, `RO-SITE-B`, and `RO-SITE-C`
- **1 Cisco provider-edge switch**: `SW-QINQ`

Each remote site is assigned its own customer VLAN and connected logically to `RO-DIST`. The provider-edge switch handles **QinQ encapsulation** and forwards the traffic through **VLAN 500** as the service transport VLAN.

### Site-to-VLAN Mapping

- **SITE-A** → VLAN 10
- **SITE-B** → VLAN 20
- **SITE-C** → VLAN 30

### Provider Service VLAN

- **Provider VLAN / S-VLAN** → VLAN 500

---

## Architecture Summary

This project follows a simple provider transport model:

- **Customer side** uses normal VLANs (`10`, `20`, `30`)
- **RO-DIST** aggregates those customer VLANs using VLAN subinterfaces
- **SW-QINQ** receives the traffic and performs **802.1Q tunneling**
- The provider side carries all customer traffic inside **VLAN 500**
- The original customer VLAN remains preserved as the **inner tag**

This makes it possible to transport multiple customer services across a single provider transport domain without redesigning the customer VLAN structure.

---

## VLAN and IP Plan

| Site | Customer VLAN | RO-DIST IP | Site Router IP | Subnet |
|------|---------------|------------|----------------|--------|
| SITE-A | 10 | 192.168.10.1/30 | 192.168.10.2/30 | 192.168.10.0/30 |
| SITE-B | 20 | 192.168.20.1/30 | 192.168.20.2/30 | 192.168.20.0/30 |
| SITE-C | 30 | 192.168.30.1/30 | 192.168.30.2/30 | 192.168.30.0/30 |

### Provider Transport VLAN

| Purpose | VLAN |
|---------|------|
| QinQ Service VLAN / Outer VLAN | 500 |

---

## Device Roles

### RO-DIST
`RO-DIST` acts as the **central distribution router** and aggregates multiple customer VLANs using subinterfaces on a single physical interface. It serves as the hub that connects to all remote sites.

Configured VLAN interfaces on `ether1`:

- `SITE-A` → VLAN 10
- `SITE-B` → VLAN 20
- `SITE-C` → VLAN 30

Assigned IP addresses:

- `192.168.10.1/30`
- `192.168.20.1/30`
- `192.168.30.1/30`

---

### RO-SITE-A / RO-SITE-B / RO-SITE-C
These routers simulate **remote branch/customer sites**. Each site is connected to `RO-DIST` through its own VLAN and /30 subnet.

- **RO-SITE-A** → VLAN 10 → `192.168.10.2/30`
- **RO-SITE-B** → VLAN 20 → `192.168.20.2/30`
- **RO-SITE-C** → VLAN 30 → `192.168.30.2/30`

Each site router creates a VLAN interface toward the provider-facing side and uses that VLAN as its transport segment.

---

### SW-QINQ
`SW-QINQ` acts as the **provider-edge switch** responsible for **QinQ encapsulation**.

Based on the configuration:

- The customer-facing interface toward **RO-DIST** is configured in **dot1q-tunnel** mode
- The customer-facing side is associated with **VLAN 500**
- The provider-facing uplink is configured as a **trunk**
- **VLAN 500** is allowed across the provider-facing side

This allows customer VLAN traffic to be carried through the provider network using an outer service VLAN while preserving the inner customer VLAN tag.

---

## How QinQ Works in This Lab

This lab uses a **double-tagging concept**:

- The **inner VLAN tag** belongs to the customer service (`10`, `20`, `30`)
- The **outer VLAN tag** belongs to the provider transport (`500`)

When traffic from a site enters the provider edge:

1. The site sends traffic using its assigned customer VLAN.
2. `RO-DIST` handles the customer VLAN as a normal VLAN subinterface.
3. The traffic is forwarded toward `SW-QINQ`.
4. `SW-QINQ` applies **QinQ tunneling**, placing the customer frame into **VLAN 500**.
5. The provider network carries the frame using the outer VLAN while keeping the original customer VLAN intact.
6. On the far side, the traffic can be mapped back to the correct customer service.

This design is commonly used in **Metro Ethernet** or **provider transport environments** where multiple customer services need to share the same infrastructure.

---

## Traffic Flow

The following example shows the traffic flow for **SITE-A**:

1. **RO-SITE-A** sends traffic on **VLAN 10**
2. The traffic reaches **RO-DIST** on its **SITE-A VLAN interface**
3. `RO-DIST` forwards the frame toward **SW-QINQ**
4. `SW-QINQ` receives the frame and places it into **provider VLAN 500**
5. The provider side transports the frame through the shared network
6. The original customer VLAN information remains preserved as the **inner tag**

The same logic applies to:

- **SITE-B** using **VLAN 20**
- **SITE-C** using **VLAN 30**

---

## Configuration Summary

### RO-DIST
Creates three VLAN interfaces on `ether1`:

- VLAN 10 → `SITE-A`
- VLAN 20 → `SITE-B`
- VLAN 30 → `SITE-C`

IP addressing:

- `192.168.10.1/30`
- `192.168.20.1/30`
- `192.168.30.1/30`

---

### RO-SITE-A
- VLAN 10 on `ether1`
- IP: `192.168.10.2/30`

### RO-SITE-B
- VLAN 20 on `ether1`
- IP: `192.168.20.2/30`

### RO-SITE-C
- VLAN 30 on `ether1`
- IP: `192.168.30.2/30`

---

### SW-QINQ
Provider-edge switch configuration concept:

- Customer-facing port → **dot1q-tunnel**
- Access / service VLAN → **500**
- Provider-facing port → **trunk**
- Allowed VLAN → **500**

---

## Verification and Testing

The expected result of this lab is that **RO-DIST** can communicate with each remote site through its assigned VLAN while the provider side carries the traffic through **VLAN 500**.

### Validation Checklist

- Verify VLAN interfaces on **RO-DIST**
- Verify VLAN configuration on each **RO-SITE**
- Check IP addressing on all routers
- Confirm **dot1q-tunnel** configuration on **SW-QINQ**
- Confirm **VLAN 500** is allowed on the provider-facing trunk
- Test connectivity using ping between **RO-DIST** and each site router

### Example Connectivity Tests

- `RO-DIST ↔ RO-SITE-A`
- `RO-DIST ↔ RO-SITE-B`
- `RO-DIST ↔ RO-SITE-C`

If all tests succeed, it indicates that:

- customer VLANs are mapped correctly,
- QinQ encapsulation is working as expected,
- the provider transport is successfully carrying the site traffic.

---

## Repository Contents

This repository contains the main configuration files used in the lab:

| File | Description |
|------|-------------|
| `ro-dist.cfg` | MikroTik configuration for the distribution router |
| `RO SITE A.cfg` | MikroTik configuration for remote site A |
| `RO SITE B.cfg` | MikroTik configuration for remote site B |
| `RO SITE C.cfg` | MikroTik configuration for remote site C |
| `SW-QINQ.cfg` | Cisco switch configuration for QinQ transport |

---

## Suggested Screenshots

To make the repository more complete and easier to understand, you can add screenshots such as:

- **Topology diagram**
- **QinQ transport path**
- **dot1q-tunnel configuration on SW-QINQ**
- **switch trunk configuration**
- **RO-DIST VLAN interfaces**
- **RO-SITE IP addressing**
- **ping verification from RO-DIST to remote sites**

Example markdown:

```md
## Topology
![Topology](images/topology.png)

## QinQ Transport
![QinQ Transport](images/transport.png)

## Switch Trunk Configuration
![Switch Trunk](images/sw-trunk.png)

## Ping Verification
![Ping Test](images/ping-ro-dist.png)
```

---

## Use Cases

This lab can be used as a reference for learning or demonstrating:

- **QinQ / 802.1Q tunneling**
- **provider edge VLAN handoff**
- **transporting multiple customer VLANs across a shared provider network**
- **Metro Ethernet service concepts**
- **basic service delivery over provider transport**
- **MikroTik and Cisco interoperability in a lab environment**

---

## Conclusion

This project demonstrates a practical implementation of **QinQ transport** using **MikroTik routers** and a **Cisco provider-edge switch**. By separating customer services into **VLAN 10**, **VLAN 20**, and **VLAN 30**, and carrying them through **provider VLAN 500**, the lab shows how multiple customer services can be transported across a shared infrastructure while maintaining VLAN separation.

Although simplified for lab purposes, the design reflects a real provider concept where customer VLANs are preserved across the backbone using a service VLAN. It is a useful project for understanding **QinQ service delivery**, **provider VLAN tagging**, and **site-to-site service transport** in an enterprise or ISP context.

---

## Notes

- This lab is focused on **QinQ service transport logic**, not on full production-grade MPLS service orchestration.
- The IP plan shown in this README follows the logical design of the topology:
  - VLAN 10 → `192.168.10.0/30`
  - VLAN 20 → `192.168.20.0/30`
  - VLAN 30 → `192.168.30.0/30`

If any configuration file contains a typo in the `network` field, the addressing table in this README reflects the intended design rather than the typo.

---

## Author

**Feri Pratama**  
Network Engineer | ISP / NOC | Routing & Switching | MikroTik & Cisco Lab
