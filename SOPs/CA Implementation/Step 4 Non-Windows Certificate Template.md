[View PDF Version](<./PDF Step 4 Non-Windows Certificate Template.pdf>)

# Purpose

Step-by-step procedure on creating Certificate Templates that will be compatible with our non-Windows devices of varying capabilities.

# Scope

#### In-Scope

- Creation of Certificate Template with 2048-bit key and SHA 256 for Non-Windows devices
- Creation of Certificate Template with 2048-bit key and SHA 1 for Non-Windows devices
- Creation of Certificate Template with 1024-bit key and SHA 1 for Non-Windows devices

#### Out-of-Scope

- Manual install of certificates on non-domain joined or non-Windows devices

# Requirements

- Permissions
  - Domain Administrator on Intermediate CA
- Remote Access to Intermediate CA server

# Procedure

## \[High-Security\] Creation of SHA256-2048bit Non-Windows Certificate Template

**In the Intermediate CA server**

- Open **Certification Authority (certsrv.msc)**
- Under our server, go to **Certificate Templates**
- Right click **Certificate Templates** and click **Manage**
- Right-click **Web Server** and click **Duplicate Template**
- In the **General** tab,
  - Name the template as necessary. I suggest **SHA256-2048 NonWindows Authentication**
  - Set the **Validity Period** as necessary (preferably **2-3 years**)

Some web browsers might reject if validity is longer than 398 days, if so change the Validity Period to 1 year

- In the **Compatibility** tab,
  - Set **Certification Authority** as **Windows Server 2008 R2**
  - Set **Certification Recipient** as **Windows 7 / Server 2008 R2**
- In the **Subject Name** tab,
  - Select **Supply in the request**
- In the **Cryptography** tab,
  - Set **Provider Category** as **Legacy Cryptographic Service Provider**
  - Set **Algorithm Name** as **RSA**
  - Set **Minimum Key Size** as **2048**
  - Check **Requests must use one of the following providers:**
  - For **Providers**, check **Microsoft RSA Schannel Cryptographic Provider**
  - For **Request hash**, set to **SHA256**
- In the **Extension** tab,
  - For **Application Policies**, ensure **Server Authentication** is listed
  - For **Key Usage**, ensure **Digital Signature** and **Key Encipherment** are listed
- In the **Request Handling** tab,
  - Set **Purpose** to **Signature and encryption**
  - Uncheck **Allow private key to be exported** (for security)
- In the **Security** tab,
  - For any group, such as Domain Admins, that will be allowed to enroll/issue certs on InterCA, ensure the permissions, **Read** and **Enroll** are checked to **Allow**

## \[Med-Security\] Creation of SHA1-2048bit Non-Windows Certificate Template

**In the Intermediate CA server**

- Open **Certification Authority (certsrv.msc)**
- Under our server, go to **Certificate Templates**
- Right click **Certificate Templates** and click **Manage**
- Right-click **Web Server** and click **Duplicate Template**
- In the **General** tab,
  - Name the template as necessary. I suggest **SHA1-2048 NonWindows Authentication**
  - Set the **Validity Period** as necessary (preferably **2-3 years**)

Some web browsers might reject if validity is longer than 398 days, if so change the Validity Period to 1 year

- In the **Compatibility** tab,
  - Set **Certification Authority** as **Windows Server 2008 R2**
  - Set **Certification Recipient** as **Windows 7 / Server 2008 R2**
- In the **Subject Name** tab,
  - Select **Supply in the request**
- In the **Cryptography** tab,
  - Set **Provider Category** as **Legacy Cryptographic Service Provider**
  - Set **Algorithm Name** as **RSA**
  - Set **Minimum Key Size** as **2048**
  - Check **Requests must use one of the following providers:**
  - For **Providers**, check **Microsoft RSA Schannel Cryptographic Provider**
  - For **Request hash**, set to **SHA1**
- In the **Extension** tab,
  - For **Application Policies**, ensure **Server Authentication** is listed
  - For **Key Usage**, ensure **Digital Signature** and **Key Encipherment** are listed
- In the **Request Handling** tab,
  - Set **Purpose** to **Signature and encryption**
  - Uncheck **Allow private key to be exported** (for security)
- In the **Security** tab,
  - For any group, such as Domain Admins, that will be allowed to enroll/issue certs on InterCA, ensure the permissions, **Read** and **Enroll** are checked to **Allow**

## \[Low-Security\] Creation of SHA1-1024bit Non-Windows Certificate Template

**In the Intermediate CA server**

- Open **Certification Authority (certsrv.msc)**
- Under our server, go to **Certificate Templates**
- Right click **Certificate Templates** and click **Manage**
- Right-click **Web Server** and click **Duplicate Template**
- In the **General** tab,
  - Name the template as necessary. I suggest **SHA1-1024 NonWindows Authentication**
  - Set the **Validity Period** as necessary (preferably **2-3 years**)

Some web browsers might alert if validity is longer than 398 days, if so change the Validity Period to 1 year

- In the **Compatibility** tab,
  - Set **Certification Authority** as **Windows Server 2003**
  - Set **Certification Recipient** as **Windows XP / Server 2003**
- In the **Subject Name** tab,
  - Select **Supply in the request**
- In the **Cryptography** tab,
  - Set **Provider Category** as **Legacy Cryptographic Service Provider**
  - Set **Algorithm Name** as **RSA**
  - Set **Minimum Key Size** as **1024**
  - Check **Requests must use one of the following providers:**
  - For **Providers**, check **Microsoft Strong Cryptographic Provider**
  - For **Request hash**, set to **SHA1**
- In the **Extension** tab,
  - For **Application Policies**, ensure **Server Authentication** is listed
  - For **Key Usage**, ensure **Digital Signature** and **Key Encipherment** are listed
- In the **Request Handling** tab,
  - Set **Purpose** to **Signature and encryption**
  - Uncheck **Allow private key to be exported** (for security)
- In the **Security** tab,
  - For any group, such as Domain Admins, that will be allowed to enroll/issue certs on InterCA, ensure the permissions, **Read** and **Enroll** are checked to **Allow**

## Issuing the newly created Certificate Template

**This step needs to be done for each of the three templates created**

- In **Certificate Authority (certsrv.msc)**, under our server, right-click **Certificate Templates**, and click **New > Certificate Template to Issue**
- In the **Enable Certificate Templates**, select the created certificate template and click **Ok**