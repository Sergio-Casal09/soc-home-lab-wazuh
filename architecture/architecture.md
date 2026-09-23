# SOC Home Lab — Architecture

## Overview

The laboratory is an isolated SOC environment built with VirtualBox.

The environment consists of three virtual machines connected through an internal
VirtualBox network named `SOC-LAB`.

The lab combines:

- Active Directory
- Windows 11
- Sysmon
- PowerShell logging
- Wazuh Agent
- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard

The objective is to collect Windows security telemetry, process it through Wazuh,
apply custom detection rules and visualize the resulting alerts.

---

## Network

```text
Network: SOC-LAB
Subnet: 192.168.10.0/24
```

The internal interfaces do not use a gateway.

Internet connectivity is provided separately through the VirtualBox NAT adapter.

---

## Virtual Machines

| Host | Sistema | IP | Función |
|---|---|---|---|
| DC01 | Windows Server 2022 | 192.168.10.10 | Active Directory + DNS |
| CLIENT01 | Windows 11 | 192.168.10.20 | Endpoint monitorizado |
| WAZUH01 | Ubuntu Server 24.04 | 192.168.10.30 | Wazuh Manager + Indexer + Dashboard |

---

## Logical Architecture

```text

                         INTERNET
                            │
                           NAT
                            │
                ┌───────────┴───────────┐
                │       VirtualBox      │
                │                       │
                │      SOC-LAB          │
                │  192.168.10.0/24      │
                │                       │
                │   ┌───────────────┐   │
                │   │ DC01          │   │
                │   │ Windows       │   │
                │   │ Server 2022   │   │
                │   │               │   │
                │   │ AD DS + DNS   │   │
                │   │ 192.168.10.10 │   │
                │   └───────┬───────┘   │
                │           │           │
                │           │           │
                │   ┌───────▼───────┐   │
                │   │ CLIENT01      │   │
                │   │ Windows 11    │   │
                │   │               │   │
                │   │ Sysmon        │   │
                │   │ PowerShell    │   │
                │   │ Wazuh Agent   │   │
                │   │               │   │
                │   │ 192.168.10.20 │   │
                │   └───────┬───────┘   │
                │           │           │
                │           │ Telemetry │
                │           ▼           │
                │   ┌───────────────┐   │
                │   │ WAZUH01       │   │
                │   │ Ubuntu 24.04  │   │
                │   │               │   │
                │   │ Wazuh Manager │   │
                │   │ Wazuh Indexer │   │
                │   │ Dashboard     │   │
                │   │               │   │
                │   │ 192.168.10.30 │   │
                │   └───────────────┘   │
                │                       │
                └───────────────────────┘

```

## Active Directory

The Active Directory domain is:

```text
soclab.local
```
The domain controller provides:

- Active Directory Domain Services
- DNS
- Global Catalog  

Organizational Units:

```text
SOC-LAB
├── Users
├── Groups
├── Workstations
└── Servers
```





















