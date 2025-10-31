# -SOC-Visibility-Dashboard
Splunk-based SOC Visibility Dashboard that analyzes Windows Security and Sysmon logs to visualize endpoint and authentication activities.
1. Overview

The SOC Visibility Dashboard is a Splunk-based project designed to simulate a Security Operations Center (SOC) environment.
It ingests and analyzes Windows Security and Sysmon logs to visualize key security activities, including authentication events, privilege escalations, credential access, and log distribution patterns.

2. Tools and Environment
| Component                     | Description                                                       |
| ----------------------------- | ----------------------------------------------------------------- |
| Splunk Enterprise (Localhost) | SIEM platform used for indexing and visualizing event data        |
| Sysmon (System Monitor)       | Windows telemetry tool for detailed system and process monitoring |
| Windows Event Logs            | Security event source (e.g., Event ID 4624, 4672, 4907, 5379)     |
| Custom Index                  | Data indexed under `soclab` for analysis                          |

3. Data Sources
| Source Type                                      | Description                                | File               |
| ------------------------------------------------ | ------------------------------------------ | ------------------ |
| WinEventLog:Microsoft-Windows-Sysmon/Operational | Sysmon endpoint activity logs              | `sysmon_log.evtx`  |
| xml_security                                     | Exported Windows Security log (XML format) | `security_log.xml` |

4. Dashboard Panels
| Panel Title                                         | Purpose                                                                    | Query Summary               | Chart Type |
| --------------------------------------------------- | -------------------------------------------------------------------------- | --------------------------- | ---------- |
| Top Windows Security Events by Frequency            | Identifies the most frequent security events by Event ID                   | `stats count by EventID`    | Bar        |
| Successful User Logons (Event ID 4624)              | Tracks user logon activity and detects abnormal login patterns             | `search EventID=4624`       | Line       |
| Privileged Logons (Event ID 4672)                   | Highlights high-privilege or administrator account logons                  | `search EventID=4672`       | Line       |
| Permission and Policy Modifications (Event ID 4907) | Monitors policy or permission changes to detect potential insider threats  | `search EventID=4907`       | Line       |
| Credential Access Activity by Host (Event ID 5379)  | Shows credential read attempts to identify potential credential theft      | `search EventID=5379`       | Pie        |
| Security Events Overview by Category                | Groups key events (4624, 4672, 4688, etc.) to show trends by activity type | `eval EventLabel=case(...)` | Line       |
| Event Distribution: Sysmon vs Windows Security      | Compares data ingestion coverage between Sysmon and Security logs          | `stats count by sourcetype` | Pie        |

5. Key Use Cases
. Detect unusual spikes in logon or authentication activity.
. Identify administrator-level privilege escalations.
. Monitor permission or security policy changes.
. Track credential access attempts across endpoints.
. Validate data visibility between Sysmon and Security log sources.

6. Example Queries
   a. Top Event IDs
   index=soclab sourcetype="xml_security"
   | rex "<EventID>(?<EventID>\d+)</EventID>"
   | stats count by EventID
   | sort -count
   b. Credential Access Events
   index=soclab sourcetype="xml_security"
   | rex "<EventID>(?<EventID>\d+)</EventID>"
   | search EventID=5379
   | stats count by host
   c. Combined SOC Overview
   index=soclab sourcetype="xml_security"
   | rex "<EventID>(?<EventID>\d+)</EventID>"
   | eval EventLabel=case(
       EventID=4624, "Logon Success",
       EventID=4672, "Admin Logon",
       EventID=4688, "Process Creation",
       EventID=4907, "Permission Change",
       EventID=5379, "Credential Read",
       1=1, "Other")
   | timechart count by EventLabel span=1h
