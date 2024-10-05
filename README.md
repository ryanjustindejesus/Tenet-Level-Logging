<h1>Tenet Level Logging</h1>
<b>This tutorial outlines the configuration of Azure Active Directory and auditlogs </b>

<h2>Environments and Technologies Used</h2>

- <b>Microsoft Azure</b> 
- <b>Azure Active Directory</b>
- <b>Log Analytics Workspace</b>

<h2>Operating Systems</h2>

- <b>Windows 10</b>

<h2>Configuration Steps</h2>

![image](https://github.com/user-attachments/assets/25451b45-f456-426f-ad64-ce86c4ec3913)
- <b>Navigate to Azure Active Directory and click diagnostic settings</b>
- <b>Add diagnostic setting</b>

![image](https://github.com/user-attachments/assets/b02ecf90-940a-4136-be88-776823519a39)
- <b>Select AuditLogs and SigninLogs</b>
- <b>Select Send to Log Analytics workspace</b>
- <b>Subscription: Azure Subscription 1</b>
- <b>Log Analytics Workspace: LAW-Cyber-Lab-01 (eastus2)</b>
- <b>Click save</b>

![image](https://github.com/user-attachments/assets/15b586a3-29ff-4ceb-8a3a-a398bd7bfe73)
- <b>Navigate to Azure Active Directory</b>
- <b>Create a dummy user</b>

![image](https://github.com/user-attachments/assets/dbc8540e-9312-4b9e-9561-e7d346abb087)
- <b>Click dummy user and select assigned roles</b>
- <b>Assign Global Administrator</b>

![image](https://github.com/user-attachments/assets/de86cf2b-5d86-45e5-98e5-69d4009552b2)
- <b>Navigate to Log Analytics Workspace and observe the audit logs using this query:
``` 
AuditLogs
| order by TimeGenerated desc
```
![image](https://github.com/user-attachments/assets/8c7aed26-3446-4897-9dc9-d66f3ad8b99e)
``` 
AuditLogs
| where OperationName == "Add member to role" and Result == "success"
| where TargetResources[0].modifiedProperties[1].newValue == '"Global Administrator"' or TargetResources[0].modifiedProperties[1].newValue == '"Company Administrator"' 
| order by TimeGenerated desc
| project TimeGenerated, OperationName, AssignedRole = TargetResources[0].modifiedProperties[1].newValue, Status = Result, TargetResources
```

![image](https://github.com/user-attachments/assets/f7e8bfb4-7644-484b-86a4-6ccea5673c4b)
- <b>Create a new user named attacker</b>

![image](https://github.com/user-attachments/assets/0e62f4a7-1dc7-445b-b4ae-a4c45e6d6098)
- <b>Perform multiple failed login attempts as the attack user in Azure

![image](https://github.com/user-attachments/assets/b27edab0-43b6-4900-8c1b-a56e1ad94999)
- <b>Sign in as the attack user in Azure</b>

![image](https://github.com/user-attachments/assets/8191f51b-debf-4cf2-a73e-cf1690a96a49)
- <b>Navigate to Log Analytics Workspace and observe the failed login attempts using this query:
``` 
SigninLogs
| where ResultDescription == "Invalid username or password or Invalid on-premise username or password."
| extend location = parse_json(LocationDetails)
| extend City = location.city, State = location.state, Country = location.countryOrRegion, Latitude = location.geoCoordinates.latitude, Longitude = location.geoCoordinates.longitude
| project TimeGenerated, ResultDescription, UserPrincipalName, AppDisplayName, IPAddress, IPAddressFromResourceProvider, City, State, Country, Latitude, Longitude

```
