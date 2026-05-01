[[View PDF Version]](<./SOPs/CA Implementation/PDF Step 5 Non-Windows CA Distribution.pdf>)

# Purpose

"Standardized" step-by-step procedure for requesting, signing, and installing certificates on non-windows devices using an Active Directory Certificate Services (AD CS) environment

# Scope

#### In-Scope

- Generation of Certificate Signing Request (CSR) on third-party hardware
- Signing CSR with the Intermediate CA
- Importing signed Certificate back to third-party hardware

#### Out-of-Scope

- Steps specific to any vendor-specific hardware

# Requirements

- Permissions
  - Access to \[Intermediate CA Server\] user account with **Read** and **Enroll** permissions for templates in Certificate Authority
  - Privileged account on non-windows allowed to configure "Security" settings
- Files: **.cer/.crt** for both Offline Root and Online Intermediate CA (if upload is possible)

# Procedure

## Importing Certificate Authority certificates into Non-Windows Device

**Must be done before importing any signed CSR**

- With **privileged account**, log into the (web) management interface of target device
- Navigate to **Security** or **Certificate** section of interface
- **If available**, begin target device's process to **configure/set-up/import** a **CA Certificate**
- When importing,
  - **If only one CA Certificate can be imported**, import the Intermediate CA certificate
  - **If more than one CA Certificate can be imported**
    - **First** import the Root CA certificate
    - **Second** import the Intermediate CA certificate

Certificate Authority certificates (**.crt/.cer**) can be found in **\\\\\[Intermediate CA Server\]\\CertEnroll\\**

## Generating the Certificate Signing Request on Non-Windows Device

- With **privileged account**, log into the (web) management interface of target device
- Navigate to **Security** or **Certificate** section of interface
- **If possible,** consider changing the **Hash Algorithm** and the **Key Length** to fit our strongest certificate template (**SHA256 / 2048-bits**)
  - Upgrading **Hash Algorithm** from SHA1 to SHA256 might bring a slight increase in time for a TLS handshake
  - Upgrading **Key Length** from 1024 to 2048 bits might bring initial connection "lag" due to 8x computational expense
- Select the option to **configure/set-up** the device's certificate
  - If provided with more options, select **Create Certificate Request** or **Create Certificate Signing Request**
- When providing additional information for the request,
  - Ensure **Common Name** and **Subject Alternative Name** is the device's **fully qualified domain name (FQDN)**
  - **All other metadata is ignored** by our CAs but you may set them as necessary
  - This may be where you can change the hash and key length
  - Click **Continue** or **Next**
- After the certificate signing request has been created, save and rename the **.csr** file as necessary

## Signing Certificate Signing Request Methods

### Signing the .csr with the Intermediate CA Server's Web Enrollment Portal

**The .csr file must be base-64 encoded**

- On a browser, go to [http://\[Intermediate_Server_FQDN\]/certsrv/](http://interca.scada.local/certsrv/)
- Authenticate as a user with **Enroll** permissions set for the certificate template to be used
- Go to **Request a certificate >** **advanced certificate request**
- For **Saved Request,** paste the content of our targeted **.csr** file
- For **Certificate Template,** select the certificate template that is compatible with the **.csr** file
- **If unable to set Subject Alternative Name during .csr creation**
  - For **Additional Attributes**, enter **san:dns=**FQDN**&ipaddress=**IP
- Click **Submit**

### Signing the .csr through CLI on remote computer

**Must be a domain user that has Enroll permissions on template and allowed RPC to Intermediate CA server**

- Open command prompt (cmd.exe)
- Navigate (**cd**) to directory containing the targeted **.csr** file
- **Run** (with adjusted template name)
  - **If unable to set Subject Alternative Name during .csr creation**, add to the following command:
    - **\-attrib "SAN:dns=**FQDN&ipaddress=IP**"**
  - **certreq -submit -attrib "CertificateTemplate:**TemplateName**"** fileexample.csr
- In **Certification Authority List** window, select **\[Intermediate CA Name\]**
- If submission successful, save and rename the **.cer/.crt** file as necessary

## Importing Signed Certificate onto back to Non-Windows Device

- With **privileged account**, log into the (web) management interface of target device
- Navigate to **Security** or **Certificate** section of interface
- Select option to **Import Signed Certificate / Upload Certificate / Install Certificate**
  - These options may also be found in the **Configure Certificate** section of interface
- Upload the appropriate signed **.cer/.crt** file for the target device
  - If available, uncheck **Mark private key as exportable** or something along those lines
- If prompted, click **Apply / Save / Restart Network Services**
  - **Web server may restart to apply the new certificate**

## Verification

- Reopen browser to clear any old cached certificate errors
- Navigate to target device's (web) management interface
- Click the lock icon in the address bar
- Ensure certificate details match our CA environment as well as the hash algorithm and key length for the chosen template