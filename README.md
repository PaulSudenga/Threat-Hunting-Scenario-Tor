<img width="400" src="https://github.com/user-attachments/assets/44bac428-01bb-4fe9-9d85-96cba7698bee" alt="Tor Logo with the onion and a crosshair on it"/>

# Threat Hunt Report: Unauthorized TOR Usage
- [Scenario Creation](https://github.com/PaulSudenga/Threat-Hunting-Scenario-Tor/blob/main/threat-hunting-scenario-tor-event-creation.md)

## Platforms and Languages Leveraged
- Windows 10 Virtual Machines (Microsoft Azure)
- EDR Platform: Microsoft Defender for Endpoint
- Kusto Query Language (KQL)
- Tor Browser

##  Scenario

Management suspects that some employees may be using TOR browsers to bypass network security controls because recent network logs show unusual encrypted traffic patterns and connections to known TOR entry nodes. Additionally, there have been anonymous reports of employees discussing ways to access restricted sites during work hours. The goal is to detect any TOR usage and analyze related security incidents to mitigate potential risks. If any use of TOR is found, notify management.

### High-Level TOR-Related IoC Discovery Plan

- **Check `DeviceFileEvents`** for any `tor(.exe)` or `firefox(.exe)` file events.
- **Check `DeviceProcessEvents`** for any signs of installation or usage.
- **Check `DeviceNetworkEvents`** for any signs of outgoing connections over known TOR ports.

---

## Steps Taken

### 1. Searched the `DeviceFileEvents` Table

Searched DeviceFileEvents table for any file that had the string “tor” in it and discovered what looks like the user “CyberScy” downloaded a tor installer, did something that resulted in many tor-related files being copied to the desktop and creation of a file called `tor-shopping-list.txt` on the desktop at `2025-06-11T21:39:21.583459Z`.  These events began at `2025-06-11T21:09:53.1315901Z`.

**Query used to locate events:**

```kql
DeviceFileEvents
| where DeviceName == "cyberchamber"
| where InitiatingProcessAccountName == "cyberscy"
| where FileName contains "tor"
| where Timestamp >= datetime(2025-06-11T21:09:53.1315901Z)
| order by Timestamp desc
| project Timestamp, DeviceName, ActionType, FileName, FolderPath, SHA256, Account = InitiatingProcessAccountName

```
![image](https://github.com/user-attachments/assets/98b9d32e-2697-40b3-a8e3-854affbe6152)


---

### 2. Searched the `DeviceProcessEvents` Table

Searched the DeviceProcessEvents table for any `ProcessCommandLine` that contained the string “"tor-browser-windows-x86_64-portable-14.5.3".  Based on the logs returned, At `2:15:35 PM on June 11, 2025`, an employee on the "cyberchamber" device ran the file `tor‑browser‑windows‑x86_64‑portable‑14.5.3.exe`, stored in CyberScy’s Downloads folder, using a command that triggered a silent installation.


**Query used to locate event:**

```kql

DeviceProcessEvents
| where DeviceName == "cyberchamber"
| where ProcessCommandLine contains "tor-browser-windows-x86_64-portable-14.5.3"
| project Timestamp, DeviceName, ActionType, FileName, FolderPath, SHA256, ProcessCommandLine

```
![image](https://github.com/user-attachments/assets/2eacbd73-ede2-4c17-a9dc-e40d546a8d4f)


---

### 3. Searched the `DeviceProcessEvents` Table for TOR Browser Execution

Searched the DeviceProcessEvents tab for any indication that user “CyberScy” actually opened the tor browser.  There was evidence that they did open it at `2025-06-11T21:16:30.9247503Z`
There were several other instances of `firefox.exe` (Tor) as well as `tor.exe` spawned afterwards.

**Query used to locate events:**

```kql
DeviceProcessEvents
| where DeviceName == "cyberchamber"
| where FileName has_any ("tor.exe", "firefox.exe", "tor-browser.exe")
| project Timestamp, DeviceName, AccountName, ActionType, FileName, FolderPath, SHA256, ProcessCommandLine
| order by Timestamp desc
```
![image](https://github.com/user-attachments/assets/e52cdc03-008b-4440-a096-bf87ea44a568)


---

### 4. Searched the `DeviceNetworkEvents` Table for TOR Network Connections

Searched the deviceNetworkEvents table for any indication the tor browser was used to establish a connection using any of the known tor ports.  At `2:17:05 PM on June 11, 2025`, an employee on the “cyberchamber” device successfully established a connection to the remote IP address `184.174.38.53` on port `9001`.  The connection was initiated by the process `tor.exe` launched by user “cyberscy” from their Desktop folder under `C:\users\cyberscy\desktop\tor browser\browser\torbrowser\tor\tor.exe`.  There were a few other connections to sites over other ports such as `443`, and `9150`.


**Query used to locate events:**

```kql
DeviceNetworkEvents
| where DeviceName == "cyberchamber"
| where  InitiatingProcessAccountName != "system"
| where InitiatingProcessFileName in ("tor.exe", "firefox.exe")
| where RemotePort in ("9001", "9030", "9050", "9051", "9150", "443", "80")
| project Timestamp, DeviceName, InitiatingProcessAccountName, ActionType, RemotePort, RemoteIP, InitiatingProcessFileName, InitiatingProcessFolderPath
| order by Timestamp desc
```
![image](https://github.com/user-attachments/assets/1c8c0e2a-b917-4586-8634-4ca16c17461c)


---

## Chronological Event Timeline 

### 1. File Download - TOR Installer

- **Timestamp:** `2025‑06‑11T21:09:53.1315901Z`
- **Event:** The user "cyberscy" downloaded a file named `tor-browser-windows-x86_64-portable-14.5.3.exe` to the Downloads folder.
- **Action:** File download detected.
- **File Path:** `C:\Users\cyberscy\Downloads\tor-browser-windows-x86_64-portable-14.5.3.exe`

### 2. Process Execution - TOR Browser Installation

- **Timestamp:** `2025‑06‑11T21:39:21.583459Z`
- **Event:** The user "cyberscy" executed the file `tor-browser-windows-x86_64-portable-14.5.3.exe` in silent mode, initiating a background installation of the TOR Browser.
- **Action:** Process creation detected.
- **Command:** `tor-browser-windows-x86_64-portable-14.5.3 /S`
- **File Path:** `C:\Users\cyberscy\Downloads\tor-browser-windows-x86_64-portable-14.5.3.exe`

### 3. Process Execution - TOR Browser Launch

- **Timestamp:** `2025‑06‑11T14:15:35 PDT (21:15:35 UTC)`
- **Event:** User "cyberscy" opened the TOR browser. Subsequent processes associated with TOR browser, such as `firefox.exe` and `tor.exe`, were also created, indicating that the browser launched successfully.
- **Action:** Process creation of TOR browser-related executables detected.
- **File Path:** `C:\Users\cyberscy\Desktop\Tor Browser\Browser\TorBrowser\Tor\tor.exe`

### 4. Network Connection - TOR Network

- **Timestamp:** `2025‑06‑11T14:16:30.9247503 PDT (21:16:30.9247503 UTC)`
- **Event:** A network connection to IP `184.174.38.53` on port `9001` by user "cyberscy" was established using `tor.exe`, confirming TOR browser network activity.
- **Action:** Connection success.
- **Process:** `tor.exe`
- **File Path:** `c:\users\cyberscy\desktop\tor browser\browser\torbrowser\tor\tor.exe`

### 5. Additional Network Connections - TOR Browser Activity

- **Timestamps:**
  - `2025-06-11T21:17:33.8919962Z` - Connected to `37.46.211.6` on port `443`.
  - `2025-06-11T21:17:29.2435404Z` - Local connection to `127.0.0.1` on port `9150`.
- **Event:** Additional TOR network connections were established, indicating ongoing activity by user "cyberscy" through the TOR browser.
- **Action:** Multiple successful connections detected.

### 6. File Creation - TOR Shopping List

- **Timestamp:** `2025-06-11T21:39:21.583459Z`
- **Event:** The user "cyberscy" created a file named `tor-shopping-list.txt` on the desktop, potentially indicating a list or notes related to their TOR browser activities.
- **Action:** File creation detected.
- **File Path:** `C:\Users\cyberscy\Desktop\tor-shopping-list.txt`

---

## Summary

The user "cyberscy" on the "cyberchamber" device initiated and completed the installation of the TOR browser. They proceeded to launch the browser, establish connections within the TOR network, and created various files related to TOR on their desktop, including a file named `tor-shopping-list.txt`. This sequence of activities indicates that the user actively installed, configured, and used the TOR browser, likely for anonymous browsing purposes, with possible documentation in the form of the "shopping list" file.

---

## Response Taken

TOR usage was confirmed on the endpoint `cyberchamber` by the user `cyberscy`. The device was isolated, and the user's direct manager was notified.

---
