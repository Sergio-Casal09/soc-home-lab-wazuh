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
CLIENT01 is located inside:

```text
SOC-LAB
└── Workstations
```
--- 

## Windows Telemetry

CLIENT01 is the main monitored endpoint.

The endpoint generates telemetry through:

### Windows Security Events

Authentication activity is monitored through Windows Security Events,
including Event ID:

```text
4625
```
### PowerShell

The following logging mechanisms are enabled:

- Script Block Logging
- Module Logging
- PowerShell Transcription

PowerShell Script Block events are collected from:

```text
Microsoft-Windows-PowerShell/Operational
```
### Sysmon

Sysmon provides process and system telemetry.

The laboratory uses events including:

```text
Event ID 1  → Process Create
Event ID 3  → Network Connection
Event ID 11 → File Create
Event ID 22 → DNS Query
```
Sysmon events are collected from:

```text
Microsoft-Windows-Sysmon/Operational
```
---

## Wazuh Telemetry Flow

The telemetry flow is:

```text
Windows Endpoint
       │
       ├── Security Events
       │
       ├── PowerShell
       │
       └── Sysmon
             │
             ▼
        Wazuh Agent
             │
             ▼
       Wazuh Manager
             │
             ▼
      Detection Rules
             │
             ▼
         Alerts
             │
             ▼
      Wazuh Dashboard
```

## Detection Layer

Custom Wazuh rules are used to detect specific activity.

The laboratory contains the following custom rules:

| Rule | Detection |
|---|---|
| 100100 | Windows failed logon |
| 100101 | Possible password guessing |
| 100110 | Suspicious PowerShell |
| 100121 | PowerShell → cmd.exe |
| 100123 | PowerShell → sensitive system utility |

---
## MITRE ATT&CK Mapping

The validated detections include:

| Technique | Description |
|---|---|
| T1059.001 | PowerShell |
| T1059.003 | Windows Command Shell |
| T1110.001 | Password Guessing |

---

## Detection Examples
### PowerShell

```text
PowerShell
    │
    └── ScriptBlock Event 4104
            │
            ▼
        Rule 100110
            │
            ▼
      T1059.001
```
### PowerShell → cmd.exe

```text
PowerShell
    │
    ▼
  cmd.exe
    │
    ▼
  whoami
```
Detected by:

```text
Rule 100121
```
### PowerShell → secedit.exe

```text
PowerShell
    │
    ▼
secedit.exe
```

Detected by:

```text
Rule 100123
```

---

### Dashboard

The Wazuh Dashboard provides a centralized view of the generated alerts.

The laboratory dashboard is:

```text
SOC LAB - Windows Detection Overview
```
It contains:

- Alertas por nivel de severidad
- Alertas de PowerShell sospechoso
- Cadenas de ejecución detectadas

---

## Security Model

The laboratory is designed as an isolated environment.

The internal network is separated from the external network and the virtual
machines are used exclusively for controlled security testing.

No production credentials or sensitive data are required by the project.

---

## Project Flow

The complete laboratory workflow is:

```text
Infrastructure
     ↓
Active Directory
     ↓
Windows Endpoint
     ↓
Sysmon + PowerShell Logging
     ↓
Wazuh Agent
     ↓
Wazuh Manager
     ↓
Custom Detection Rules
     ↓
MITRE ATT&CK Mapping
     ↓
Wazuh Alerts
     ↓
Dashboard
     ↓
Evidence
```

