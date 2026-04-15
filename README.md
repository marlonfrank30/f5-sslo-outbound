# F5 SSLO Outbound Architecture

This repo contains a full production-quality SSLO documentation including flows, macros, service chains, and design insights.

---

# 🧭 Architecture Overview

```mermaid
flowchart LR
A[Clients] --> B[SSLO]
B --> C[Decrypt]
C --> D[Service Chain]
D --> E[Re-encrypt]
E --> F[Internet]
```

![SSLO Outbound Network Diagram](./diagrams/sslo-netdiagram.png)

# 🧭 Architecture Overview with Flows
---
![SSLO Outbound Network Diagram](./diagrams/sslo-netdiagram2.png)


# 🔁 End-to-End Traffic Flow

```mermaid
flowchart TD
A[Client Request] --> B[Intercept Source Policy]
B -->|Match| C[Intercept]
B -->|No Match| Z[Bypass]

C --> D[SSL Check]
D -->|TLS| E[Decrypt]
D -->|Non-TLS| F[L7 Detection]

E --> G[Categorization]
F --> G

G --> H[Service Chain]

H --> I[FireEye]
H --> J[Firepower]
H --> K[LightSpeed]

I --> L[Return]
J --> L
K --> L

L --> M[Re-encrypt]
M --> N[Internet]

Z --> N
```

---

# 🧠 APM Policy Flow

```mermaid
flowchart TD
A[Session Check]
A -->|Exists| B[Pinners Rule]
A -->|New| C[IP Protocol Lookup]

C -->|TCP| D[L7 Lookup]
C -->|Other| Z[Reject]

D -->|HTTP/HTTPS| E[SSLO Redirect]
D -->|Other| Z

E --> F[Intercept Macro]
F --> G[Categorization Macro]
G --> H[TPS Exclusions]
H --> I[Service Chain Macro]

I --> J[FireEye]
I --> K[Firepower]
I --> L[LightSpeed]

J --> M[Allow]
K --> M
L --> M
```

---

# 🔬 Macro-Level Breakdown

## Intercept Source Macro
- Matches defined internal subnets
- Non-match = bypass

---

## Categorization Macro
IP address spreedsheet mappings <br>
![SSLO Outbound IP address assigments ](./assets/f5-IP-mappings.xlsx)

## Categorization Macro

```mermaid
flowchart TD
A[SSL Check] -->|SSL| B[Category Lookup]
A -->|No SSL| C[L7 Protocol]

C --> D[HTTP Handling]
C --> E[CONNECT Handling]

B --> F[Assign Variables]
D --> F
E --> F

F --> G[Exit]
```

---

## Service Chain Macro (sslosc_all_services)

```mermaid
flowchart LR
A[Traffic] --> B[FireEye]
A --> C[Firepower]
A --> D[LightSpeed]

B --> E[Return]
C --> E
D --> E
```

---

## Pinners Rule
- Detects certificate pinning
- Forces bypass

---

## TPS Exclusions
- Category-based bypass
- Reduces inspection load

---

# 🌐 IP Addressing (Cleaned & Grouped)

## Core Interfaces

| Function | IP | VLAN |
|---------|----|------|
| Internal | 10.4.0.8 | int-vlan |
| External | 192.168.100.13 | ext-vlan |

---

## Firepower

| Direction | IP | VLAN |
|----------|----|------|
| To | 192.168.200.225 | firepower_to |
| From | 192.168.200.241 | firepower_from |

---

## FireEye

| Direction | Range | VLAN | RD |
|----------|------|------|----|
| To | 198.19.34.0/27 | fireeye_to | 65020 |
| From | 198.19.34.0/27 | fireeye_from | 65020 |

---

## LightSpeed

| Direction | Range | VLAN | RD |
|----------|------|------|----|
| To | 198.19.35.0/27 | lightspeed_to | 65030 |
| From | 198.19.35.0/27 | lightspeed_from | 65030 |

---

# 🔀 VLAN Segmentation Strategy

```mermaid
flowchart LR
A[int-vlan] --> B[SSLO]
B --> C[ext-vlan]

B --> D[fireeye_to]
D --> E[fireeye_from]

B --> F[firepower_to]
F --> G[firepower_from]

B --> H[lightspeed_to]
H --> I[lightspeed_from]
```

---

# 🧱 Design Highlights

- Selective interception
- Full TLS visibility
- Multi-service inspection
- APM-driven decisions
- High availability (HA pair)

---

# 📌 Notes
- Built from real SSLO config
- Designed for SA-level documentation
------------------------------------------------------------------------
- ## License

This project is intended for operational automation within F5 environments.  
Use at your own risk and validate in a test environment prior to production deployment.

------------------------------------------------------------------------

# 🤝 Contributing

Pull requests welcome:

-   Automation examples
-   Architecture diagrams
-   Policy templates

------------------------------------------------------------------------

# ⭐ Credits

Architecture based on enterprise F5 BIG-IP SSLo deployment patterns and
real-world designs.

------------------------------------------------------------------------

## 🧑‍💻 Author
**Marlon Frank**  
*Network and Application Security & F5 Automation Engineer*  
