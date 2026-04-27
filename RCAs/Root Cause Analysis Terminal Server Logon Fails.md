\# Incident Description



FortiSIEM constantly triggers alerts for \*\*Sudden Increase in Failed Logons\*\* (Event 4625) on terminal servers (Most prevalent \\\[Terminal Server NetBIOS Name\\]\*\*)\*\*:



\- Account Name: \\\[Terminal Server NetBIOS Name\\]

\- Failed Logon Substatus: 0xC0000064 (User doesn't exist locally on \\\[Terminal Server NetBIOS Name\\])

\- Source Network Address: ::1 (IPv6 loopback, occurs even if IPv6 disabled)



\# Background



\## Role of Remote Desktop Connection Broker



Remote Desktop Connection Broker (tssdis.exe) is to manage user session and more relevantly, ensure session persistence.



RDCB maintains a database of active RDP sessions, so if the user is disconnected, RDCB will re-route the user back to their existing session instead of starting a new session.



RDCB's syncing process occurs each time a user authenticates into a server through RDP



\- The frequency of the alert is directly related to the server's RDP frequency (\\\[Terminal Server 1 NetBIOS Name\\] is most frequent unlike \\\[Terminal Server 2 NetBIOS Name\\])



When an RDP request is received for a server, RDCB send a network login to the Broker Database to retrieve any existing RDP sessions, if available.



\*\*Current terminal servers have an all-in-one configuration where both RDCB and Broker Database are on the same server.\*\*



\## Windows Loopback Prevention Policy



As referenced in this Microsoft Article: \[Error message when you try to access a server locally by using its FQDN or its CNAME alias after you install Windows Server 2003 Service Pack 1 - Windows Server | Microsoft Learn](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/accessing-server-locally-with-fqdn-cname-alias-denied)



Policy introduced after Microsoft Windows Server 2003 Service Pack and is enabled by default



\*\*Purpose:\*\* To prevent NTLM Reflection Attacks (MiTM). It ensures a service cannot authenticate against itself to escalate privileges. \*\*Forces Local Security Authority (LSA) to verify if target name matches the "local" NetBIOS name of computer\*\*



\### The Prevented MiTM Attack Vector (MS08-68 Case)



\_As referenced by Microsoft:\_ \[\_Microsoft Security Bulletin MS08-068 - Important | Microsoft Learn\_](https://learn.microsoft.com/en-us/security-updates/SecurityBulletins/2008/ms08-068)



The following is the standard attack process for loopback NTLM Reflection through SMB leading to RCE:



The initial service connection



\- Attacker gets initial low-privilege foothold on server and starts a "listener"

\- Attacker tricks SMB (a high privilege service) into connecting by telling SMB to check \[\\\\\\\\\*\*server.domain.local\*\*\\\\EvilShare](file:///\\server.domain.local\\EvilShare) (Due to FQDN use, server thinks it's talking to a legitimate local "Network Resource")

\- As SMB is attempting to open the share, it connects to the listener and authenticates as SYSTEM with the Attacker



The reflection (While SMB connection to Attacker is open)



\- Attacker starts an authentication request to the LSA

\- LSA generates an NTLM Challenge and sends to Attacker

\- Attacker reflects NTLM Challenge to the SMB

\- SMB (running as SYSTEM) takes the Challenge, encrypts it using its own keys, and sends response back to Attacker

\- Attacker takes the encrypted Challenge and sends it back to the LSA.

\- Since the Challenge was encrypted using keys for a SYSTEM account, LSA grants the Attacker a full SYSTEM token.



Loopback Prevention would stop this by automatically blocking any loopback logons addressed to an FQDN instead of a local name.



\# Technical Cause of Logon Failures (Event 4625)



Because RDCB and Broker database are located on the same server, the network authentication login from RDCB to Broker database is sent as a IPv6 loopback address (::1)



\- RDCB (tssdis.exe) runs as NETWORK SERVICE and attempts to log on as \\\[INTERNAL-ORG\\]\\\\\\\[Terminal Server NetBIOS Name\\]\\$ targeting the FQDN \\\[Terminal Server FQDN\\] (Shown through Event 4648)

\- LSA sees loopback request targeting FQDN \\\[Terminal Server FQDN\\] instead of a local name

\- Due to Loopback Protection, LSA fails to recognize \\\[Terminal Server FQDN\\] as a local alias

\- To continue NTLM authentication, LSA "strips" as \\\[INTERNAL-ORG\\]\\\\\\\[Terminal Server NetBIOS Name\\]\\$ and attempts to authenticate a local user \\\[Terminal Server NetBIOS Name\\]

\- Since there is no local user "\\\[Terminal Server NetBIOS Name\\]" on \\\[Terminal Server NetBIOS Name\\], the logon fails (Event 4625)



\*\*Note: If tssdis.exe was coded to connect to \\\[Terminal Server NetBIOS Name\\] (NetBIOS name) instead of \\\[Terminal Server FQDN\\] (FQDN), this error wouldn't happen\*\*



\# Possible Solutions



\## Temporary Fix and Concerns



\[Forums](https://techcommunity.microsoft.com/discussions/windowsserver/login-failure-from-tssdis-exe-on-rds-server/3567587) on this same issue (Event 4648 from tssdis leading to multiple Event 4625) have reported \*\*restarting the service tssdis.exe\*\* temporarily stops the logon fails from an hour up to weeks



Done manually or automated, however automation can lead to conflicts if multiple users successfully authenticate to server through RDP at the same time



\- One user's logon triggers the automated service restart, blocking the other users' logon attempts while service is restarting



\## Permanent Fix and Concerns



\_As referenced in this Microsoft article:\_ \[\_Error message when you try to access a server locally by using its FQDN or its CNAME alias after you install Windows Server 2003 Service Pack 1 - Windows Server | Microsoft Learn\_](https://learn.microsoft.com/en-us/troubleshoot/windows-server/networking/accessing-server-locally-with-fqdn-cname-alias-denied)



With the primary issue being LSA not identifying the FQDN as itself during the logon attempts, a possible solution is adding an "alias" that LSA can use to recognize that the FQDN is mapped to the NETBIOS name.



This can be done by editing the \*\*BackConnectionHostNames\*\* registry key on the servers:



\*\*(Requires privileges)\*\*



\- On Registry Editor, navigate to HKEY\_LOCAL\_MACHINE\\\\SYSTEM\\\\CurrentControlSet\\\\Control\\\\Lsa\\\\MSV1\_0

\- Within the folder, create a new multi-string value key named "\*\*BackConnectionHostNames"\*\*

\- Edit the value to be the FQDN \\\[Terminal Server FQDN\\] on one line and the NetBIOS name on another line

\- To apply changes, the Remote Desktop Connection Broker service needs to be restarted. It won't disrupt current sessions but will stop new sessions while down.



With these changes, any loopback logons addressed to the server's FQDN will be understood as its NetBIOS name. \*\*This is the only use of this registry key, it has no other operational effect\*\*



\### Security Risks



To understand the risks of whitelisting a FQDN for loopback network logons, it would help to understand the \[attack vector of loopback network logons (MS08-68)](#\_The\_Prevented\_MiTM)



For a loopback NTLM reflection attack on our terminal servers:



\- Attacker needs to gain an initial foothold on server (low privileges)

\- Attacker needs to know the FQDN (figured by checking Hostname and DNS Suffix)

\- Due to Service Principal Name validation, Attacker needs to connect to a service that has the appropriate NTLM packet service class that matches with the "System/Login" service of LSA

\- Due to Extended Protection for Authentication, Attacker will need to forge the unique Channel Binding Token that matches a unique NTLM handshake.

&#x20; - As soon as attacker reflects the CBT of a service onto LSA, the CBTs won't match and be rejected



\### Operational Impact



This change only impacts Windows-level authentication, specifically any loopback logons as they automatically default to NTLM authentication. Due to this scope, there is no expected effect on any \\\[INTERNAL-ORG\\] operation



This change doesn't remove anything from the registry editor, all it does is whitelist the FQDN as an acceptable target machine name for network loopback logons.



\# Conclusion



Since Loopback Policy had been implemented in 2008, current security measures such as SPN and EPA are in place to prevent any forging and reflection of authentication requests.



With the whitelist of a FQDN, an attacker can try to leverage a service with SYSTEM access for NTLM reflection but these new security measures make it incredibly difficult.



This issue has been bringing clusters of alerts whenever any user successfully authenticates into terminal server through RDP. With all this noise, it would be beneficial to mitigate the noise at the server-level rather than the SIEM-level

