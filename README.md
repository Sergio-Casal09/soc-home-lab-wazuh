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
```
---

### Virtual Machines

| Host | Sistema | IP | Función |
|---|---|---|---|
| DC01 | Windows Server 2022 | 192.168.10.10 | Active Directory + DNS |
| CLIENT01 | Windows 11 | 192.168.10.20 | Endpoint monitorizado |
| WAZUH01 | Ubuntu Server 24.04 | 192.168.10.30 | Wazuh Manager + Indexer + Dashboard |

### Dominio

```text
soclab.local
```
### Red interna

```text
SOC-LAB
192.168.10.0/24
```
---

### 🔧 Tecnologías utilizadas


- VirtualBox
- Windows Server 2022
- Windows 11
- Ubuntu Server 24.04
- Active Directory Domain Services
- DNS
- Group Policy
- PowerShell Logging
- Sysmon
- Wazuh Agent
- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- MITRE ATT&CK

---

### 🏢 Active Directory
El dominio utilizado en el laboratorio es:

```text
soclab.local
```
### Estructura de Active Directory

```text
SOC-LAB
├── Users
├── Groups
├── Workstations
└── Servers
```
### Usuarios
- alice
- bob
- analyst
### Grupos
- SOC-Analysts
- IT-Admins
- Helpdesk

CLIENT01 se encuentra dentro de:

```text
SOC-LAB
└── Workstations
```
### Group Policy
Se configuró la GPO:

```text
GPO-Workstations-Security
```
La auditoría avanzada incluye:

- Logon — Success + Failure
- Account Lockout — Success + Failure

---

### 🔎 Windows Telemetry

### PowerShell Logging

Se habilitaron:

- PowerShell Script Block Logging
- PowerShell Module Logging
- PowerShell Transcription

Esto permite obtener visibilidad sobre la ejecución de scripts PowerShell.

### Sysmon

Sysmon se instaló en CLIENT01 para obtener telemetría de:

- creación de procesos
- conexiones de red
- carga de imágenes
- creación de archivos
- modificaciones de registro
- consultas DNS
- eliminación de archivos
- actividad de procesos

### Eventos Sysmon utilizados

| Event ID | Descripción |
|---|---|
| 1 | Process Create |
| 3 | Network Connection |
| 11 | File Create |
| 22 | DNS Query |

---

### 🛡️ Wazuh

CLIENT01 ejecuta Wazuh Agent y envía eventos al servidor:

```text
WAZUH01
192.168.10.30
```

Se configuró la recogida de:

```text
Microsoft-Windows-Sysmon/Operational
```
y:

```text
Microsoft-Windows-PowerShell/Operational
```
### Flujo de detección

```text
Windows
   │
   ├── Security Events
   ├── PowerShell
   └── Sysmon
          │
          ▼
     Wazuh Agent
          │
          ▼
     Wazuh Manager
          │
          ▼
    Custom Rules
          │
          ▼
     Wazuh Alerts
          │
          ▼
   Wazuh Dashboard
```
---

### 🚨 Detecciones personalizadas

### 100100 — Windows Failed Logon

Detecta fallos de autenticación Windows mediante:

```text
Security Event ID 4625
```
Nivel: 5

---

### 100101 — Possible Password Guessing

Correlaciona:

```text
5 fallos de autenticación
```
para el mismo usuario dentro de:

```text
60 segundos
```

#### MITRE ATT&CK:

```text
T1110.001 — Password Guessing
```
---

### 100110 — Suspicious PowerShell

Detecta determinados patrones dentro de:

```text
PowerShell ScriptBlock
Event ID 4104
```

Patrones utilizados:

```text
FromBase64String
Invoke-Expression
IEX
```

#### MITRE ATT&CK:

```text
T1059.001 — PowerShell
```

#### Prueba realizada

```text
[Convert]::FromBase64String("SGVsbG8=")
```

#### Resultado

```text
Wazuh Rule 100110
Level 10
MITRE T1059.001
```

---

### 100121 — PowerShell → cmd.exe

Detecta una cadena de ejecución donde:

```text
ParentImage = PowerShell
```
y:

```text
Image = cmd.exe
```

#### MITRE ATT&CK

```text
T1059.001 — PowerShell
T1059.003 — Windows Command Shell
```

#### Prueba realizada

```text
cmd.exe /c whoami
```
#### Cadena observada

```text
PowerShell → cmd.exe → whoami
```
---

### 100123 — PowerShell → Sensitive Utility

Detecta PowerShell ejecutando determinadas herramientas sensibles del sistema.

Herramientas contempladas:

```text
secedit.exe
certutil.exe
nltest.exe
bitsadmin.exe
```

#### Prueba realizada

```text
powershell.exe -Command "secedit /export /cfg C:\Windows\Temp\secpol-test.cfg"
```
#### Cadena observada
```text
PowerShell → secedit.exe
```
#### MITRE ATT&CK

```text
T1059.001 — PowerShell
```

---

### 🧪 Validación de detecciones

Las detecciones fueron probadas mediante actividad controlada dentro del laboratorio.


| Detección | Evento | Regla | Resultado |
|---|---|---|---|
| Failed Logon | 4625 | 100100 | Validada |
| Password Guessing | 4625 | 100101 | Validada |
| Suspicious PowerShell | 4104 | 100110 | Validada |
| PowerShell → cmd.exe | Sysmon 1 | 100121 | Validada |
| PowerShell → secedit | Sysmon 1 | 100123 | Validada |

---











