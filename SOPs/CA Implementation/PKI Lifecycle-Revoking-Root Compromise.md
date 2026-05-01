[[View PDF Version]](<./SOPS/CA Implementation/PDF PKI Lifecycle-Revoking-Root Compromise.pdf>)

# Purpose

Standard procedure for Root and Intermediate CA Renewal and Certificate Revocation Updates. Additionally, Revoking Procedures at the bottom.

**Table for Reference**

| PKI Component                    | Validity Period | Last Renewal | Expiration Date<br><br>(Next Renewal) |
| -------------------------------- | --------------- | ------------ | ------------------------------------- |
| Root Certificate                 | X Years         |              |                                       |
| Root Certificate Revocation List | X Years         |              |                                       |
| Intermediate Certificate         | X Years         |              |                                       |

**We should revoke a certificate if:**

- Device is **stolen** (Attacker can pull private key from storage)
- Device is **compromised** (Attacker can borrow "identity" for any purpose)
- Device is **retired** (Attacker can scavenge private key and use a "legacy" identity)
- Certificate Authority is **stolen/compromised/retired** (Attacker mimics CA)

# Scope

- Renewing Certificate for Root Certificate Authority **(Every 10 years)**
- Renewing Certificate Revocation List for Root Certificate Authority **(Every 5 years)**
- Renewing Certificate for Intermediate Certificate Authority **(Every 5 years)**
- Revoking certificates and pushing out the new certificate revocation list **(Ad-hoc)**

# Procedure

**Follow procedure based on the Maintenance Event (5 years / 10 years)**

## Routine Maintenance (Every 5 Years)

### (1) Renewing the Root Certificate Revocation List

**On Root CA Server**

- In **Certificate Authority (certsrv.msc),**
  - Right Click **Revoked Certificates >** Go to **All Task** > Click **Publish**
- Transfer the new **.crl** file onto a secure media

(found in **C:\\Windows\\System32\\CertSrv\\CertEnroll)**

### (2) Renewing the Intermediate CA Certificate / Transferring the Root CRL

**On Intermediate CA Server**

- In **Certificate Authority (certsrv.msc),**
  - Right Click **CA** **Name** > Go to **All Tasks** \> Click **Renew CA Certificate**
- **In the pop-up box, do not generate a new private/public key**
- Save the **.req** file onto a secure media
- From secure media, transfer the lastest root **.crl** file into C**:\\Windows\\System32\\CertSrv\\CertEnroll**
- Right-click the **.crl** file and select **Install**.
- Set the **Certificate Store** as **Intermediate Certification Authorities**, finish wizard with defaults
- In **C:\\inetpub\\wwwroot\\CertEnroll**, replace the old root **.crl** file with the new file. **The name must be the same.**

### (3) Signing the Intermediate CSR / Installing the Signed CSR

**On Root CA Server**

- Mount secure media containing the **.req** file
- Copy the latest **.req** file onto **C:\\**
- In **Certificate Authority**, right-click the **CA Name** \> go to **All Tasks** > click **Submit new request…**
- In the **Open Request File** window, open the latest imported **.req** file
- In **Certificate Authority**, go to **Pending Requests** > **right-click** the latest request > go to **All Tasks** > and click **Issue**
- Go to **Issued Certificates >** **right-click** the latest certificate, and click **Open**
- In the **Certificate** window, go to **Details** tab and **click Copy to File…**
- In **Certificate Export Wizard**, set the format to be **Base-64 encoded X.509 (.CER)** (this format should be readable for all SCADA device)
- Continue with defaults, **Save** file name as **\[domain_IntermediateCAName_CA\].crt**
- Transfer the latest Intermediate CA **.crt** file onto secure media

**On Intermediate CA Server**

- Mount secure media containing latest Intermediate CA .**cer/.crt** file
- In **Certificate Authority**, right-click the **CA Name** > go to **All Tasks** > select **Install CA Certificate**
- Open the latest Intermediate CA **.cer/.crt** file
- In **C:\\inetpub\\wwwroot\\CertEnroll**,
  - Replace the old Intermediate CA **.cer/.crt** file with the latest (name must be the same)

**Restart the Certificate Services service**

## Complete Maintenance (Every 10 Years)

### (1) Renewing the Root CA Certificate

**On Root CA Server**

- In **Certificate Authority**, right-click **CA Name** > go to **All Tasks** > select **Renew CA Certificate…**
- Stop Active Directory Certificate Services to continue
- **In the pop-up box, do not generate a new private/public key**
- Under **C:\\Windows\\System32\\CertSrv\\CertEnroll**, transfer the latest root **.cer/.crt** file onto secure media
  - **Must replace all instances of old root certificate with latest**

### (2) Renewing the Root Certificate Revocation List

**On Root CA Server**

- In **Certificate Authority (certsrv.msc),**
  - Right Click **Revoked Certificates >** Go to **All Task** > Click **Publish**
- Transfer the new **.crl** file onto a secure media

(found in **C:\\Windows\\System32\\CertSrv\\CertEnroll)**

### (3) Transferring the Root Cert and CRL / Renewing Intermediate CA Certificate

**On Intermediate CA Server**

- From secure media, transfer the lastest root **.crl** and **.crt/.cer** files into C**:\\Windows\\System32\\CertSrv\\CertEnroll**
- Right-click the root **.crt/.cer** file and select **Install**
- Set the **Certificate Store** as **Trusted Root Certification Authorities**, finish wizard with defaults
- Right-click the **.crl** file and select **Install**.
- Set the **Certificate Store** as **Intermediate Certification Authorities**, finish wizard with defaults
- In **C:\\inetpub\\wwwroot\\CertEnroll**, replace the old root **.crl** and **.crt/.cer** files with the new files. **The name must be the same or else logic filepath fails.**
- In **Certificate Authority (certsrv.msc),**
  - Right Click **CA** **Name** > Go to **All Tasks** \> Click **Renew CA Certificate**
- **In the pop-up box, do not generate a new private/public key**
- Save the **.req** file onto a secure media

### (4) Signing the Intermediate CSR / Installing the Signed CSR

**On Root CA Server**

- Mount secure media containing the **.req** file
- Copy the latest **.req** file onto **C:\\**
- In **Certificate Authority**, right-click the **CA Name** \> go to **All Tasks** > click **Submit new request…**
- In the **Open Request File** window, open the latest imported **.req** file
- In **Certificate Authority**, go to **Pending Requests** > **right-click** the latest request > go to **All Tasks** > and click **Issue**
- Go to **Issued Certificates >** **right-click** the latest certificate, and click **Open**
- In the **Certificate** window, go to **Details** tab and **click Copy to File…**
- In **Certificate Export Wizard**, set the format to be **Base-64 encoded X.509 (.CER)** (this format should be readable for all SCADA device)
- Continue with defaults, **Save** file name as **\[domain_IntermediateCAName_CA\].crt**
- Transfer the latest InterCA **.crt** file onto secure media

**On Intermediate CA Server**

- Mount secure media containing latest InterCA .**cer/.crt** file
- In **Certificate Authority**, right-click the **CA Name** > go to **All Tasks** > select **Install CA Certificate**
- Open the latest Intermediate CA **.cer/.crt** file
- In **C:\\inetpub\\wwwroot\\CertEnroll**,
  - Replace the old Intermediate CA Name **.cer/.crt** file with the latest (name must be the same)

**Restart the Certificate Services service**

## Revoking Certificates (Ad-Hoc Basis)

### Revoking A Child Device's Certificate

**On Intermediate CA Server**

- In **Certificate Authority**, go to **Issued Certificates** > right-click the **target certificate** > select **Revoke Certificate**
- Right-click **Revoked Certificates** \> **All Tasks** \> **Publish** and choose **New CRL**

**New .crl file in C:\\Windows\\System32\\CertSrv\\CertEnroll**

- In **C:\\inetpub\\wwwroot\\CertEnroll**, replace the old **.crl** file with latest one (**Names must match)**

**Windows: As of 4/30/2026, our Intermediate CA is set to auto-update CRL every X days. If wish to immediately update device's CRL you must run "certutil -urlcache \* delete" on every device**

**Non-Windows: Some allow installation of CRL, if so, you must manually install on all capable devices**

### Revoking Intermediate CA's Certificate

**Since the Intermediate CA server also hosted the IIS webserver, the whole Intermediate CA component needs to be rebuilt**

- Power and logon to the Offline Root CA server
- In **Certificate Authority**, go to **Issued Certificates** > right-click the **Intermediate CA certificate** > **All Tasks** > select **Revoke Certificate**
- Right-click **Revoked Certificates** \> **All Tasks** \> **Publish** and choose **New CRL**

**New .crl file in C:\\Windows\\System32\\CertSrv\\CertEnroll**

- Follow **SOP - Step 2 Online Intermediate CA Creation**
  - **Remember to import the new Root CRL**
  - **Must have same FQDN**
- Follow **SOP - Step 3 Domain CA Distribution with Auto Enrollment**
  - For section _Importing the Root CA and Intermediate CA in the GPO_, **only need to import the Intermediate CA certificate**
  - Each
- Follow **SOP - Step 4 Non-Windows Certificate Template**
- To recertify devices as well as push new CRL,
  - **For Windows,** previously "certified" devices need to run the following:
    - **gpupdate /force** (to get new certs)
    - **certutil -urlcache \* delete** (to forget old CA)
  - **For Non-Windows,** previously "certified" devices need a new manual certification

## Disaster Recovery During Root Compromise

**If server is breached or private key is stolen, entire PKI environment must be rebuilt**

- Power down / wipe the Root and/or Intermediate CA servers
- Follow **SOP - Step 1 Offline Root CA Creation**
- Follow **SOP - Step 2 Online Intermediate CA Creation**
- On **Domain Controller,** right-click our "PKI" GPO > select **Edit**
  - Go to **Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies**
  - For **Trusted Root Certification Authorities**, remove the old Root CA certificate
  - For **Intermediate Certification Authorities**, remove the old Intermediate CA certificate
- Follow **SOP - Step 3 Domain CA Distribution with Auto Enrollment**
  - Skip section _Creating the GPO in Domain Controller_
  - Skip section _Enabling Auto-Enrollment Policy in the GPO_
- Follow **SOP - Step 4 Non-Windows Certificate Template**
- Follow **SOP - Step 5 Non-Windows CA Distribution**