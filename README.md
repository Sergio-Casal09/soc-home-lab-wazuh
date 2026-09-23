# SOC Home Lab — Active Directory, Windows, Sysmon & Wazuh

![Windows Server](https://img.shields.io/badge/Windows%20Server-2022-blue)
![Windows 11](https://img.shields.io/badge/Windows-11-blue)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04-orange)
![Wazuh](https://img.shields.io/badge/Wazuh-4.14-red)
![Sysmon](https://img.shields.io/badge/Sysmon-Sysinternals-purple)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE%20ATT%26CK-Mapped-yellow)

Laboratorio personal de operaciones de seguridad (SOC) construido para practicar
monitorización, detección y análisis de eventos de seguridad en un entorno
Windows/Active Directory.

---

## 🎯 Objetivo

El objetivo del proyecto es construir un entorno SOC aislado en VirtualBox
capaz de:

- Monitorizar endpoints Windows.
- Recoger eventos de seguridad mediante Wazuh.
- Obtener telemetría de procesos mediante Sysmon.
- Registrar actividad de PowerShell.
- Detectar fallos de autenticación.
- Crear reglas de detección personalizadas.
- Mapear detecciones a MITRE ATT&CK.
- Analizar cadenas de ejecución de procesos.
- Visualizar las alertas mediante un dashboard SOC.

---

## 🏗️ Arquitectura

```text
                         INTERNET
                            │
                         NAT
                            │
              ┌─────────────┴─────────────┐
              │       VirtualBox          │
              │                           │
              │      SOC-LAB Network      │
              │     192.168.10.0/24       │
              │                           │
              │  ┌─────────────────────┐  │
              │  │ DC01                │  │
              │  │ Windows Server 2022 │  │
              │  │ 192.168.10.10       │  │
              │  │ AD DS + DNS         │  │
              │  └──────────┬──────────┘  │
              │             │             │
              │  ┌──────────▼──────────┐  │
              │  │ CLIENT01            │  │
              │  │ Windows 11          │  │
              │  │ 192.168.10.20       │  │
              │  │ Sysmon + Wazuh      │  │
              │  │ Agent               │  │
              │  └──────────┬──────────┘  │
              │             │             │
              │             │ Telemetry   │
              │             ▼             │
              │  ┌─────────────────────┐  │
              │  │ WAZUH01             │  │
              │  │ Ubuntu Server 24.04 │  │
              │  │ 192.168.10.30       │  │
              │  │ Manager + Indexer   │  │
              │  │ + Dashboard         │  │
              │  └─────────────────────┘  │
              │                           │
              └───────────────────────────┘
