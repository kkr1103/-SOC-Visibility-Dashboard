# -SOC-Visibility-Dashboard
Splunk-based SOC Visibility Dashboard that analyzes Windows Security and Sysmon logs to visualize endpoint and authentication activities.
1. Overview
The SOC Visibility Dashboard is a Splunk-based project developed to simulate a Security Operations Center (SOC) environment. It analyzes Windows Security and Sysmon logs to visualize authentication activities, privilege escalations, credential access, and policy changes. The project demonstrates how a SOC can detect endpoint and user activity trends using Splunk’s data analysis and visualization capabilities.

2. Tools and Environment
The following components were used to build and run the SOC Visibility Dashboard:
. Splunk Enterprise (Localhost): Used as the SIEM platform for indexing, searching, and visualizing event data.
. Sysmon (System Monitor): Provides detailed telemetry of system and process activity on Windows machines.
. Windows Event Logs: Supplies core security events such as Event IDs 4624 (Logon Success), 4672 (Admin Logon), 4907 (Permission Change), and 5379 (Credential Access).
. Custom Index – soclab: A dedicated index created in Splunk for ingesting and analyzing log data.

3. Data Sources
Two main log sources were ingested into Splunk:
. Sysmon Operational Logs: Captured from
  WinEventLog:Microsoft-Windows-Sysmon/Operational — stored as sysmon_log.evtx.
. Windows Security Logs: Exported in XML format from Event Viewer — stored as security_log.xml.

4. Dashboard Panels
The dashboard includes multiple analytical panels, each designed to visualize specific SOC metrics:
. Top Windows Security Events by Frequency – Identifies the most frequently occurring Event IDs to reveal key activities.
. Successful User Logons (Event ID 4624) – Displays normal user authentication patterns and anomalies.
. Privileged Logons (Event ID 4672) – Highlights administrative or service account access attempts.
. Permission and Policy Modifications (Event ID 4907) – Detects security policy and descriptor changes.
. Credential Access by Host (Event ID 5379) – Monitors credential reading or dumping activity.
. Security Events Overview by Category – Groups multiple event types for trend visualization.
. Event Distribution by Source Type – Compares the volume of logs from Sysmon versus Windows Security.

5. Key Use Cases
. Detect spikes in authentication and logon events.
. Identify administrator or service account privilege escalations.
. Track security policy and permission modifications.
. Detect credential harvesting or access attempts.
. Verify coverage between endpoint (Sysmon) and Windows Security logs.

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
