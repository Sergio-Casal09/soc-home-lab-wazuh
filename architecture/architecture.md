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

