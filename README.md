# Sysmon

[![License](https://img.shields.io/badge/License-MIT-green)](https://github.com/richardhicks/sysmon/blob/main/LICENSE)

Targeted Sysmon configuration files for monitoring Microsoft Active Directory Certificate Services (AD CS) infrastructure.

## Description

This repository contains System Monitor (Sysmon) configuration files designed to provide high-signal security monitoring for Active Directory Certificate Services (AD CS) servers. Each configuration targets a specific AD CS server role and focuses on the processes, files, registry keys, network connections, and credential access patterns that matter most for detecting attacks against certificate infrastructure.

These configurations are targeted, not general enterprise baselines. They can be deployed standalone on AD CS servers or merged into an existing baseline such as [SwiftOnSecurity/sysmon-config](https://github.com/SwiftOnSecurity/sysmon-config) or [olafhartong/sysmon-modular](https://github.com/olafhartong/sysmon-modular) as an additional rule set.

## Installation

1. Download [Sysmon](https://learn.microsoft.com/sysinternals/downloads/sysmon) from Microsoft Sysinternals.
2. Download the configuration file for the target server role from the [GitHub repository](https://github.com/richardhicks/sysmon).
3. Confirm the schema version supported by the installed Sysmon binary:

```
sysmon64.exe -s
```

4. Install Sysmon with the configuration file:

```
sysmon64.exe -accepteula -i sysmon-adcs-issuing-ca.xml
```

5. To update an existing Sysmon installation with a new configuration:

```
sysmon64.exe -c sysmon-adcs-issuing-ca.xml
```

6. To verify the currently loaded configuration:

```
sysmon64.exe -c
```

Events are written to the following event log:

```
Applications and Services Logs > Microsoft > Windows > Sysmon > Operational
```

## Configurations

| File                          | Version | Target Role                     | Description                                                                                                                                                         |
| ----------------------------- | ------- | ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sysmon-adcs-issuing-ca.xml`  | 1.0     | AD CS Issuing (Subordinate) CA  | Monitors CA configuration, policy and exit modules, CA database and key material, CRL/AIA publication, CA administration tooling, and credential access            |
| `sysmon-ndes.xml`             | 1.1     | Network Device Enrollment Service (NDES) | Monitors SCEP enrollment configuration, IIS application pool control, Registration Authority (RA) certificate handling, and credential access              |

### sysmon-adcs-issuing-ca.xml

Designed for Active Directory Certificate Services issuing (subordinate) certification authority servers. If Web Enrollment, Certificate Enrollment Web Services (CES/CEP), or NDES is co-located on the CA, review the `w3wp.exe` rules and adjust as needed.

This configuration complements, but does not replace, native CA auditing. Enable CA auditing with the following commands, then restart the Active Directory Certificate Services service:

```
certutil -setreg CA\AuditFilter 127
auditpol /set /subcategory:"Certification Services" /success:enable /failure:enable
```

### sysmon-ndes.xml

Designed for Network Device Enrollment Service (NDES) servers used for SCEP certificate enrollment, including deployments supporting Microsoft Intune and Microsoft Configuration Manager.

## Event Coverage

Both configurations include rules for the following Sysmon event types:

| Event ID | Event Type                  |
| -------- | --------------------------- |
| 1        | Process Create              |
| 2        | File Creation Time Changed  |
| 3        | Network Connection          |
| 5        | Process Terminated          |
| 6        | Driver Loaded               |
| 7        | Image Loaded                |
| 8        | Create Remote Thread        |
| 9        | Raw Access Read             |
| 10       | Process Access              |
| 11       | File Create                 |
| 12-14    | Registry Event              |
| 15       | File Create Stream Hash     |
| 17-18    | Pipe Event                  |
| 19-21    | WMI Event                   |
| 25       | Process Tampering           |
| 26       | File Delete Detected        |
| 29       | File Executable Detected    |

The issuing CA configuration also includes rules for DNS Query (Event ID 22) and File Delete Archived (Event ID 23). Event ID 23 archives deleted key material and CA artifacts to the `ArchiveDirectory` before deletion completes.

## Notes

- Registry root keys appear abbreviated in the configuration files (HKLM, HKU, HKCR).
- Rules use the `contains` condition where appropriate so that both native and WOW6432Node registry paths are captured.
- After a pilot noise review, add backup agents, SIEM forwarders, and HSM client software to the relevant exclude groups.
- All configurations are detection only. No rules block process execution or file creation.

## Requirements

- Windows Server with the Active Directory Certificate Services (AD CS) role installed.
- Sysmon 15.0 or later (configuration schema version 4.90). Event ID 29 (File Executable Detected) requires Sysmon 15.0 or later.
- Administrative privileges to install Sysmon and load configuration files.

## Author

**Richard Hicks** - [Richard M. Hicks Consulting, Inc.](https://www.richardhicks.com/)

- Website: <https://www.richardhicks.com/>
- GitHub: <https://github.com/richardhicks/sysmon>
- X: [@richardhicks](https://x.com/richardhicks)

## License

This project is licensed under the [MIT License](https://github.com/richardhicks/sysmon/blob/main/LICENSE).

## Copyright

© 2026 Richard M. Hicks Consulting, Inc. All rights reserved.
