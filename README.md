# MSA_SOC
SOC Environment on Microsoft Azure

<h3>Project:</h3>
This project is both a SOC and honeynet built in Microsoft Azure to gain hands-on practical knowledge. We will take a look at attacks and also alerts from Microsoft Sentinel. The honeynet will then be hardened to prevent attacks and a variety of metrics such as the alerts triggered will be shown below. 

The guide that we will be following for this project is from Phillip: https://www.youtube.com/watch?v=mOjbD7FkUUI


<h3>Setup:</h3> Create VM with network rule that allows all inbound traffic for our honeypot.

![image](https://github.com/user-attachments/assets/d3cf8635-536c-4c44-b368-db5333ce77f8)
![image](https://github.com/user-attachments/assets/95dd643d-9c1e-4678-b2ad-9b19db246a86)

Note: Turn on VM and RDP into it to turn off Windows Defender Firewall. 

Install SQL Server Management Studio from Microsoft:

![image](https://github.com/user-attachments/assets/7ddd7c1d-e296-4383-b889-c5066a1e82bf)

After installing SSMS, connect to the server and enable login auditing for both failed and successful logins in Server Properties->Security:

![image](https://github.com/user-attachments/assets/24d127ad-26d0-4618-95b4-3f85365b88a9)

Create a Log Analytics Workspace and add Microsoft Sentinel to it:

![image](https://github.com/user-attachments/assets/db0edc37-df8a-4c84-9007-69350ae22587)

![image](https://github.com/user-attachments/assets/5efd19be-3a10-44cf-b4e3-c651164f54b1)

Add geoip (CSV file) to watchlists: 
[geoip-summarized.csv](https://github.com/user-attachments/files/18716520/geoip-summarized.csv)
![image](https://github.com/user-attachments/assets/71bfc8a2-ee53-496f-af64-d2b17297e9c2)

Created storage account that is part of the main resource group (Dandelion).
![image](https://github.com/user-attachments/assets/b5bbd90a-6019-4897-bd34-95b842af1b17)

Create flow log with storage account we created above and flow log type of NSG.

![image](https://github.com/user-attachments/assets/fdb3ae29-de1a-419d-a1c4-11b1cfecca12)

Created Windows Event Logs data collection rule for the VM we created earlier (destination is the Log Analytics Workspace we made):
![image](https://github.com/user-attachments/assets/ebe3dda3-f396-4cca-8f3a-c1441aedd5d1)

This rule has two added XPath queries as shown below:
![image](https://github.com/user-attachments/assets/e5b4443b-9a66-494c-afca-c066fb0c11e8)

Microsoft-Windows-Windows Defender/Operational!\*[System[(EventID=1116 or EventID=1117)]]

Microsoft-Windows-Windows Firewall With Advanced Security/Firewall!*[System[(EventID=2003)]]

Make a diagnostic setting with AuditLogs and SignInLogs with the destination set as our Log Analytics workspace:
![image](https://github.com/user-attachments/assets/931f2ce6-c76e-43c7-8f29-2d73dee71e76)

Create another diagnostic setting with all categories for Azure logs under monitor instead of user:
![image](https://github.com/user-attachments/assets/ec2815a2-5c36-4218-91c6-097b2cca71df)

For the storage account we made, enable blob diagnostic setting (for blob storage logs):
![image](https://github.com/user-attachments/assets/b629dba3-52a7-448e-879c-a10ddc8a73d3)
![image](https://github.com/user-attachments/assets/cd810282-8119-42ff-a814-e39a760c8e10)

Created a key vault with vault access policy as permission mode. This is just to test for Azure logs:
![image](https://github.com/user-attachments/assets/88ede3f1-2843-4ef4-920a-7dc299b38dfd)

Added 3 workbooks which visualize the data from the logs
The first one shows authentication failures from trying to log in to the mssql server.
![image](https://github.com/user-attachments/assets/7586171f-63b8-485e-9fef-2ce5998479ac)

The second workbook shows NSG inbound malicious flows from the allow all rule that we created at the start.
![image](https://github.com/user-attachments/assets/db7c2c8c-798c-4437-a660-e819d3001374)

The last workbook is the rdp authencation failures from trying to rdp into the honeypot VM.
![image](https://github.com/user-attachments/assets/25ad87bd-5dad-46b8-9458-15c5b3277ea8)

Now it is time to harden the network by removing the inbound rule and turning on Windows Defender. Those are the most simple ways since we created those vulnerabilies for testing, but there are a lot of ways to harden the network by following what Microsoft Defender for Cloud recommends.
![image](https://github.com/user-attachments/assets/30b7c2ca-f9a2-44ca-9f02-2f160929ecbe)

After 2 days of leaving on the VM, we can take a look at the attack atempts and security events to see if the hardening was successful.

<h3>Results:</h3>
There were no brute force attempts through RDP or malicious flows since we made a whitelist for RDP and also blocked inbound traffic. For the MSSQL server, we implemented firewall rules that only allowed specific IPs such as our own to access the database server. 
