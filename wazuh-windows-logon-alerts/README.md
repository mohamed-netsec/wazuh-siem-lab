### WAZUH - SIEM Security Lab : ###
 
🏗️ Lab Architecture
Wazuh Manager: Centralized SIEM server (192.168.1.100)
Wazuh Agent: Installed on the Windows endpoint (client2, Agent ID: 001)
OS: RHEL Linux / Windows 10

### 1 . Windows Security Event Log via eventchannel :

✅ Key Implementation Steps
1) Enable Windows Security Audit Policy
Used secpol.msc on client2.
Path:
Local Policies → Audit Policy → Audit logon events
Enabled Failure auditing to capture failed logon attempts.
Enforced policy changes via:
```
gpupdate /force
```

Verified that Event ID 4625 appears in Event Viewer → Windows Logs → Security after a failed logon attempt.

### 2) Configure Wazuh Agent to Collect Security Events :

# Configuration file:
C:\Program Files (x86)\ossec-agent\ossec.conf
Ensured the following block exists:

<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
</localfile>


-------------photo



Avoided using a restrictive <query> (e.g., EventID = 4624 only) to prevent excluding 4625 events.
Restarted the agent after changes:
```powershell
Restart-Service -Name wazuh
```

### 3) Verify Connectivity and Event Flow :
On Windows (Agent):
Log file: C:\Program Files (x86)\ossec-agent\ossec.log
Checked for errors related to eventchannel or Security.

# -On Linux (Manager):

Confirm the agent is connected (agent connected) and events are not being dropped.

### 4) Monitor and Analyze via Wazuh Dashboard :

--- In the Wazuh Dashboard:
Threat hunting → Security events
Applied filters:

win.system.eventID:4625
agent.id:001 or agent name client2



### 📌 Important Notes
Ensure ports 1514 and 1515 are open between the Agent and Manager.
Always check ossec.log on the Agent if events are not appearing.
Use precise filters in the Dashboard to isolate events from client2.
