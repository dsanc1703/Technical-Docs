\# Purpose



This is a step-by-step procedure to build/configure the offline Root Certificate Authority and later sign the Intermediate CA's Certificate Signing Request.



In layman terms, the offline Root CA will "sign" the Intermediate CA as a "representative" within the domains. After which, further certificate signing with be done with the Intermediate CA



\# Scope



\#### In-Scope



\- Installation of AD CS role as a Standalone Root

\- Configuration of private key

\- Root Extension Setup

\- Signing the Intermediate CA's Certificate Signing Request

\- Setting validity periods of root certificate

\- Decommissioning of Root CA



\#### Out-of-Scope



\- Internal configuration of Intermediate CA or IIS Web Server setup



\# Requirements



Permissions: Local Adminstrator



Isolated dedicated VM for Windows Server that will be taken offline after complete initialization



Software: Windows Server 2019 or higher



Files: \[CAPolicy.inf](https://learn.microsoft.com/en-us/windows-server/networking/core-network-guide/cncg/server-certs/prepare-the-capolicy-inf-file#create-the-capolicyinf-file)



\*\*System must not be joined to the \\\[INTERNAL-ORG\\] domain\*\*



\# Procedure



\## Pre-Configuration



\- Log into the standalone server as Local Administrator

\- In a new file \*\*C:\\\\Windows\\\\CAPolicy.inf\*\*, have the following content:

&#x20; - You may edit the Root CA's certificate renewal period as necessary with the values:

&#x20;   - \*\*RenewalValidityPeriod\*\*

&#x20;   - \*\*RenewalValidityPeriodUnits\*\*



\\\[Version\\]



Signature="\\$Windows NT\\$"



\\\[PolicyStatementExtension\\]



Policies=InternalPolicy



\\\[InternalPolicy\\]



OID=1.2.3.4.1455.67.89.5



Notice="Legal Notice: \\\[INTERNAL-ORG\\] Root CA. Physical access necessary for modifications"



URL=<https://pki.corp.contoso.com/pki/cps.txt>



\\\[Certsrv\_Server\\]



RenewalKeyLength=4096



RenewalValidityPeriod=Years



RenewalValidityPeriodUnits=10



CRLPeriod=Years



CRLPeriodUnits=5



LoadDefaultTemplates=0



AlternateSignatureAlgorithm=1



\\\[CRLDistributionPoint\\]



\\\[AuthorityInformationAccess\\]



\## Root CA Role Installation



\- Open \*\*Server Manager -> Manage -> Add Roles and Features\*\*

\- In the \*\*Add Roles and Features Wizard\*\*, accept defaults until \*\*Server Roles\*\*

\- \*\*In Server Roles,\*\* add the role \*\*Active Directory Certificate Services\*\* and click \*\*Add Features\*\*

\- Accept all defaults and install

\- Once the installation is finished, on progress screen, click \*\*Configure Active Directory Certificate Service on the destination server\*\*



\## Post-Installation Role Configuration



\- Ensure the \*\*Credentials\*\* are for the \*\*Local Administrator account\*\*

\- For \*\*Role Services\*\*, check \*\*Certification Authority\*\*

\- For \*\*Setup Type\*\*, select \*\*Standalone CA\*\*

\- For \*\*CA Type\*\*, select \*\*Root CA\*\*

\- For \*\*Private Key\*\*, select \*\*Create a new private key\*\*

\- For \*\*Cryptography\*\*, select the key length \*\*(preferably 4096)\*\*

\- For \*\*CA Name\*\*, set the Common name as \*\*\\\[INTERNAL-ORG\\]-ROOTCA-CA\*\* (can be changed for better distinction)

\- For \*\*Validity Period\*\*, match the time to the renewal validity period listed in \*\*C:\\\\Windows\\\\CAPolicy.inf\*\*

\- For \*\*Certificate Database\*\*, leave as default

\- Confirm configuration and click \*\*Configure\*\*



\## Configuring Root CA Extensions



\- On \*\*Server Manager\*\*, go to \*\*Tools -> Certification Authority\*\*

\- \*\*Right-Click\*\* on our \*\*CN\*\* for our root CA and select \*\*Properties\*\*

\- In the \*\*Properties\*\* box, select the \*\*Extensions\*\* tab

\- With the \*\*CRL Distribution Point (CDP)\*\* extension selected, have only the following entries available:

&#x20; - The C:\\\\ entry

&#x20; - \\\[Change the FQDN\\]



<http://INTERCA.\\\[INTERNAL-ORG\\].local/CertEnroll/<CaName><CRLNameSuffix><DeltaCRLAllowed>.crl>



\- For the http entry, check \*\*Include in CRLS\*\* and \*\*Include in the CDP\*\*



\*\*WHEN APPLYING CHANGES, DO NOT RESTART SERVICES\*\*



\- With the \*\*Authority Information Access (AIA)\*\* extension selected, have only the following entries available:

&#x20; - The C:\\\\ entry

&#x20; - \\\[Change the FQDN\\]



<http://INTERCA.\\\[INTERNAL-ORG\\].local/CertEnroll/<CaName><CertificateName>.crt>



\- For the http entry, check \*\*Include in the AIA extension…\*\*



\*\*You can now restart the service when applying changes\*\*



\## Exporting the Root Certificate and Root Revocation List



The certificate should already be available, so we need to get the revocation file.



\- On \*\*certsrv, right-click Revoked Certificates\*\*, go to \*\*All Tasks\*\*, \*\*click Publish\*\*

\- Both the root certificate file (.cer) and the root revocation list file (.crl) should be found under \*\*C:\\\\Windows\\\\System32\\\\CertSrv\\\\CertEnroll\*\*

\- Transfer both files onto a secure media to later be transferred to the Intermediate CA server.



\## Signing The Intermediate CA request and Exporting



\*\*After the Intermediate CA server has been built/configured and its CSR has been exported\*\*



\- Mount the secure media containing the Intermediate CA's \*\*.req\*\* file

\- Copy over the \*\*.req\*\* file onto \*\*C:\\\\\*\*

\- Open the \*\*Certificate Authority Console (certsrv.msc)\*\*

\- \*\*IF wish to specify the validity period of the Inter CA\*\*

&#x20; - In admin CMD: certutil -setreg CA\\\\ValidityPeriodUnits (\*\*number\*\*)

&#x20; - In admin CMD: certutil -setreg CA\\\\ValidityPeriod "Years"

&#x20; - To apply changes: net stop certsvc \&\& net start certsvc

\- \*\*Right-click\*\* our \*\*CA Name\*\*, go to \*\*All Tasks\*\* and click \*\*Submit new request…\*\*

\- In the \*\*Open Request File\*\* window, search and open our imported \*\*.req\*\* file

\- In the \*\*Certificate Authority Console\*\*, go to \*\*Pending Requests\*\*, \*\*right-click\*\* the request, go to \*\*All Tasks\*\*, and click \*\*Issue\*\*

\- In the \*\*Certificate Authority Console\*\*, go to \*\*Issued Certificates\*\*, \*\*right-click\*\* the certificate, and click \*\*Open\*\*

\- In the \*\*Certificate\*\* window, go to \*\*Details\*\* tab and \*\*click Copy to File…\*\*

\- In the \*\*Certificate Export Wizard\*\*, set the format to be \*\*Base-64 encoded X.509 (.CER)\*\* (this format should be readable for all \\\[INTERNAL-ORG\\] devices)

\- Click \*\*Next\*\*

\- In the \*\*Save As\*\* pop-up, choose an easily-remembered location set file name as \*\*the CA Name of the Intermediate CA Server\*\*, click \*\*Save\*\*, and click \*\*Next\*\*

\- Click \*\*Finish\*\*

\- Open File Explorer and go to the location of the newly-exported \*\*InterCa\*\* file

\- Transfer \*\*InterCa\*\* onto secure media

\- Refer to SOP for importing the signed \*\*InterCA\*\* onto the Intermediate CA Server



\## Vaulting the Root CA Server



\*\*This should only be done after the Intermediate CA Server has successfully imported the (Root CA Signed) Certificate\*\*



\- Shut down the VM/physical machine

\- Secure the drive

&#x20; - If VM, move the virtual disk to an encrypted offline drive

&#x20; - If physical, lock it up



Vaulting is done to ensure the Root CA's private key can never be compromised which can lead to certificates becoming invalid



\## Clearing any old Certificate Data on ROOTCA



\- Revoke any issued certificates (the intermediate)

\- Delete any old \*\*.cer\*\* and \*\*.crl\*\* files

