# HoneyPot-Lab
HoneyPot Lab that was created through Cyber Range

Live Honeypot Breach: Exposed MySQL on Azure

I built an intentionally weak Windows 11 VM running MySQL in Azure, wired it into Microsoft Defender for Endpoint and Sentinel, exposed it to the internet, and investigated what real attackers did to it. Within 18 hours, two separate ransom groups broke in, stole and wiped the databases, and left Bitcoin ransom notes.


Environment
Component	Details
Host	Windows 11 VM (corpna1da) in Azure
Database	MySQL Server 8.0 with dummy company data
Endpoint	Microsoft Defender for Endpoint
Logging	MySQL general query log shipped by Azure Monitor Agent (DCR) to Log Analytics (MySQLAudit_CL)
SIEM	Microsoft Sentinel analytics rules for VM and MySQL logons
Network	NSG flow logs (NTANetAnalytics)

Intentional weaknesses: root@'%' with a weak password, MySQL bound to 0.0.0.0, a weak administrator account, Guest enabled, Windows Firewall off, and an NSG allowing all inbound traffic.

What happened
When (PDT)	Event
Oct 7, 3:15 PM	Honeypot exposed
Oct 8, 8:52 AM	Wave 1: bot logs in as root, dumps every table (4.15 MB out), drops 30 tables, leaves a ransom note. 84 seconds total
Oct 8, 12:13 PM	Mass scanner guesses the administrator password over RDP. Nothing runs
Oct 8, 4:11 PM	Wave 2: second group drops every database, leaves its own note, wipes the binary logs, revokes root and tries to shut MySQL down. 55 seconds total
Oct 9, 2:04 PM	VM isolated in Defender, NSG closed

Skills demonstrated
Building and instrumenting a cloud honeypot (Azure, MDE, AMA, DCR, Log Analytics)
Writing and tuning Sentinel analytics rules
Threat hunting with KQL across MySQLAudit_CL, DeviceLogonEvents, DeviceProcessEvents, DeviceFileEvents, DeviceNetworkEvents and NTANetAnalytics
Reconstructing an attack timeline from MySQL query logs and NSG flow data
Before and after forensic comparison of MDE investigation packages (autoruns, services, tasks, prefetch, BAM, Security event logs)
Containment and remediation: MDE isolation and NSG hardening
IOC extraction and MITRE ATT&CK mapping

Key takeaways
An internet facing database with a weak root password was compromised by two groups in under a day, with no exploit needed.
Shipping logs off the box saved the investigation. Wave 2 wiped the MySQL binary logs, but every query had already reached Log Analytics.
My MySQL detection fired on my own admin logins and missed both attacks. Tuning (localhost filtering, suppression, entity mapping) is covered in the report.
Outbound egress blocking didn't stop the data theft, because the data left as the reply to the attacker's own inbound connection.

MITRE ATT&CK
T1595.001 · T1110.001 · T1078 · T1190 · T1082 · T1213 · T1041 · T1098 · T1070 · T1485 · T1531 · T1489 · T1657

Lab completed through the Cyber Range. All data in the environment was dummy data.
