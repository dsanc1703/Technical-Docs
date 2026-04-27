\# Purpose



This is a step-by-step procedure to build/configure the online Intermediate Certificate Authority and handle all certificate tasks for the \\\[INTERAL-ORG\\] environment, utilizing the trust established by the offline Root CA



In layman terms, the online Intermediate CA will represent the offline Root CA and conduct any certificate-related tasks for our environment.



\# Scope



\#### In-Scope



\- Installation of AD CS role as an Enterprise Subordinate

\- Configuration of standardized Certification encryption for use by Windows and Non-Windows devices

\- Exporting Certification Signing Request from Intermediate CA server



\#### Out-of-Scope



\- Joining the server onto \\\[INTERNAL-ORG\\] domain



\# Requirements



Permissions: Domain/Local Adminstrator



Dedicated hardware/VM for Windows Server that will be online for certification



Software: Windows Server 2019 or higher



System must a FQDN (preferably INTERCA.\\\[INTERNAL-ORG\\].local)



\*\*System must be joined to the domain\*\*



\# Procedure



\## Intermediate CA Role Installation



\- Open \*\*Server Manager -> Manage -> Add Roles and Features\*\*

\- In the \*\*Add Roles and Features Wizard\*\*, accept defaults until \*\*Server Roles\*\*

\- \*\*In Server Roles,\*\* add the role \*\*Active Directory Certificate Services\*\* and click \*\*Add Features\*\*

\- Accept all defaults and until \*\*AD CS -> Role Services\*\*

\- In \*\*Role Services\*\*, in addition to \*\*Certificate Authority\*\*, \*\*check Certification Authority Web Enrollment\*\* and \*\*click Add Features\*\*

\- Accept all defaults and until \*\*Web Server Roles (IIS) -> Role Services\*\*

\- In \*\*Role Services\*\*, check \*\*Basic Authentication\*\* under \*\*Web Sever -> Security\*\*

\- Click Next, Confirm configurations, and Install.

\- Once the installation is finished, on progress screen, click \*\*Configure Active Directory Certificate Service on the destination server\*\*



\## Post-Installation Role Configuration



\- Ensure the \*\*Credentials\*\* are for the \*\*Domain Administrator account\*\* (following is an example)

\- For \*\*Role Services\*\*, check \*\*Certification Authority\*\* and \*\*Certification Authority Web Enrollment\*\*

\- For \*\*Setup Type\*\*, select \*\*Enterprise CA\*\*

\- For \*\*CA Type\*\*, select \*\*Subordinate CA\*\*

\- For \*\*Private Key\*\*, select \*\*Create a new private key\*\*

\- For \*\*Cryptography\*\*, select the key length \*\*(needs to be compatible with all \\\[INTERNAL-ORG\\]\*\* \*\*devices: Windows and Non-Windows /// Most likely 2048-bit)\*\*

\- For \*\*CA Name\*\*, set the Common name as \*\*\\\[INTERNAL-ORG\\]-INTERCA-CA\*\* (can be changed for better distinction but must match RootCA signing)

\- For \*\*Certificate Request\*\*, select \*\*Save a certificate request to file on the target machine\*\* and leave \*\*File name\*\* as default. (This option requires us to manually transfer the cert request from the Intermediate CA server to the Root CA server)

\- For \*\*Certificate Database\*\*, leave as default

\- Confirm configuration and hit \*\*Configure\*\*



\## Root CA Cert Installation



\- Mount the secure media containing the root certificate file (\*\*.cer\*\*) and the root revocation list file (\*\*.crl\*\*)

&#x20; - Transfer the root files to the Intermediate CA Server's file path \*\*C:\\\\Windows\\\\System32\\\\CertSrv\\\\CertEnroll\*\*

&#x20; - \*\*Right-click\*\* the \*\*.cer\*\* file and select \*\*Install Certificate\*\*

&#x20; - In \*\*Certificate Import Wizard\*\*, set the Store Location as \*\*Local Machine\*\*

&#x20; - In the UAC popup, allow \*\*Rundll32\*\* to make changes

&#x20; - Select \*\*Place all certificates in the following store\*\* and click \*\*Browse…\*\*

&#x20; - Set the Certificate Store as \*\*Trusted Root Certification Authorities\*\*

&#x20; - Click \*\*Ok\*\*, Click \*\*Next\*\*, Click \*\*Finish.\*\*

&#x20; - Repeat steps 3-8 for the \*\*.crl\*\* file (store in intermediate cert auth)



\## CertEnroll Folder Creation For Environment Certifications \& IIS Config



\- In File Explorer, go to file path \*\*C:\\\\inetpub\\\\wwwroot\*\*



\- Create a new folder \*\*CertEnroll\*\*



&#x20; - Copy over the root certificate file (.cer) and the root revocation list file (.crl) from C:\\\\Windows\\\\System32\\\\CertSrv\\\\CertEnroll



&#x20; - Ensure the CertEnroll folder has Read access for the IUSR account or Everyone



\- In IIS Manager for our server



&#x20; - \*\*Right-click\*\* the \*\*CertEnroll\*\* folder and select \*\*Convert to Application\*\*



&#x20; - \*\*Ensure .crl\*\* and \*\*.cer\*\* are registered \*\*MIME\*\* types



&#x20;   - Right-click server name and open \*\*MIME Types\*\*



&#x20;   - If no \*\*application/x-x509-ca-cert\*\* entry, add it manually



&#x20;   - If no \*\*application/pkix-crl\*\* entry, add it manually



&#x20; - Go to \*\*Request Filtering > Edit Feature\*\* Settings, and check \*\*Allow Double Escaping\*\*



\## Inbound Firewall Rule to allow PKI Traffic



\- Open \*\*Windows Defender Firewall with Advanced Security\*\*

\- \*\*Right-Click Inbound Rule\*\* and select \*\*New Rule…\*\*

\- For \*\*Rule Type\*\*, set as \*\*Port\*\*

\- Set the \*\*Specific local port\*\* as \*\*80\*\*

\- Accept all defaults and click next

\- Set the \*\*Rule Name\*\* as \*\*IIS Web Server\*\*



\## Exporting Intermediate CA Server's Certification Request for Root CA signing



\- Locate the \*\*.req\*\* file (should be in \*\*C:\\\\\*\* directory)

\- Transfer the \*\*.req\*\* file into secure media and mount the media onto the Root CA server

\- Refer to SOP for signing the Intermediate CA Server's Certification Request on the Root CA Server



\## Importing Signed Intermediate CA Server's Certification Request



\- Mount the secure media containing the signed \*\*.cer\*\* file

\- Open \*\*Certificate Authority (certsrv.msc)\*\*

\- \*\*Right-click\*\* our server, go to \*\*All Tasks\*\*, and click on \*\*Install CA Certificate…\*\*

\- In the \*\*Select file to complete CA installation\*\* pop-up, go to secure media and \*\*Open\*\* the \*\*InterCA.cer\*\* file

&#x20; - You may need to change the file type filter from \\\*.p7b

&#x20; - \*\*Ensure .cer file is also in the C:\\\\inetpub\\\\wwwroot\\\\CertEnroll folder\*\*

&#x20; - Ensure a copy of the intermediate CRL exists in the same folder as well (may have to publish the CRL to move)



\## Verification of Successful Root CA Certification Implementation



\- In \*\*Certificate Authority (certsrv.msc)\*\*, \*\*Right-click\*\* our server, go to \*\*All Tasks\*\*, and click \*\*Start Service\*\*



Green checkmark next to the server name is good sign



\- For further review of our successful certificate, \*\*right-click\*\* the server and click \*\*Properties\*\*

\- In the \*\*Properties\*\* dialog box, click \*\*View Certificate\*\* for the only available CA Certificate

\- You may review the \*\*Certificate\*\* information to confirm all fields are correct



\## Clearing any old Certificate Data on INTERCA



If server has any existing certificate data that will be overwritten



\- Run \*\*certlm.msc\*\* (Local Machine Store)

&#x20; - Delete old root from Trusted Root Certification Authorities

&#x20; - Delete old intermediate from Intermediate Certification Authorities

\- Purge the AIA/CDP URL cache

&#x20; - Run as admin: \*\*certutil -urlcache \\\* delete\*\*

\- Clean Up Web Folder

&#x20; - Delete all files in \*\*C:\\\\inetpub\\\\wwwroot\\\\CertEnroll\*\*



\## Verifying IIS connection



\- Ensure both \*\*.cer\*\* and \*\*.crl\*\* files are in \*\*C:\\\\inetpub\\\\wwwroot\\\\CertEnroll\*\*

\- On a different machine connected to \\\[INTERNAL-ORG\\], go to URLs:

&#x20; - http://\\\[IIS-Server-Name-Or-FQDN\\]/CertEnroll/\\\[Cerfile\\].cer

&#x20; - http://\\\[IIS-Server-Name-Or-FQDN\\]/CertEnroll/\\\[Crlfile\\].crl

\- Successful connectivity if both results in opening/downloading file



\## Verifying IIS validity



\- Ensure both \*\*.cer\*\* and \*\*.crl\*\* files are in \*\*C:\\\\inetpub\\\\wwwroot\\\\CertEnroll\*\*

\- Open command prompt

&#x20; - \*\*certutil -url http://\&lt;Your-IIS-Server\&gt;/CertEnroll/\&lt;Your-INTERCA-Name\&gt;.crl\*\*

\- In tool window, select \*\*URL\*\* radio button and click \*\*Retrieve\*\*

\- Successful validity if row is green and reports \*\*Verified\*\*



Next step is to create a Certificate Template on the Domain Controller. There is a SOP for this.

