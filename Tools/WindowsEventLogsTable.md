# Windows Event Logs Table

| Log Name | Provider Name  | Event IDs | Description |
| ------------- | ------------- | ------------- | ------------- |
| System || 7045 | A service was installed in the system |
| System || 7030	| ...service is marked as an interactive service.  However, the system is configured to not allow interactive services. This service may not function properly.|
| System || 1056	| Create RDP certificate |
| Security || 7045, 10000, 10001, 10100, 20001, 20002, 20003, 24576, 24577, 24579 | Insert USB |
| Security || 4624 | Account Logon |
| Security || 4625 | Failed login |
| Security || 4688 | Process creation logging |
| Security || 4720 |  A user account was created |
| Security || 4722 | A user account was enabled |
| Secutity || 4724, 4738 | Additional user creation events |
| Security || 4728 | A member was added to a security-enabled global group |
| Security || 4732| A member was added to a security-enabled local group |
| Security || 	1102 | Clear Event log |
| Application | EMET | 2 | EMET detected ... mitigation and will close the application: ...exe |
| Firewall || 2003 | Disable firewall |
| Microsoft-Windows-AppLocker/EXE and DLL ||	8003 | (EXE/MSI) was allowed to run but would have been prevented from running if the AppLocker policy were enforced |
| Microsoft-Windows-AppLocker/EXE and DLL ||	8004 | (EXE/MSI) was prevented from running. |
| Microsoft-Windows-WindowsDefender/Operational || 1116 | Windows Defender has detected malware or other potentially unwanted software|
| Microsoft-Windows-WindowsDefender/Operational || 1117 | Windows Defender has taken action to protect this machine from malware or other potentially unwanted software|
