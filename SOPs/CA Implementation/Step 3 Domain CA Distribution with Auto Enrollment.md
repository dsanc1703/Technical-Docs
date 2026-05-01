[View PDF Version](<./PDF Step 3 Domain CA Distribution with Auto Enrollment.pdf>)

# Purpose



This is a step-by-step procedure on distributing the Offline Root CA's certificate to all [INTERNAL-ORG] domain computers. This ensures **Windows** devices will automatically trust certificates from the Intermediate CA as well as receive auto-enrollment.



# Scope



#### In-Scope



- Group Policy Object to distribute Root CA certificate and auto enroll Windows devices



#### Out-of-Scope



- Manual install of certificates on non-domain joined or non-Windows devices



# Requirements



Permissions: Domain Administrator



Access to Windows Domain Controllers and Online Intermediate CA



Ensure Domain workstations/servers can reach the Online Intermediate CA server



# Procedure



## Creating the GPO in Domain Controller



- Go to **Group Policy Management (gpmc.msc)**

- **Right-click** on our **domain** and click **Create a GPO in this domain, and Link it here…**

- Set name as necessary, for clarity I recommend **[INTERNAL-ORG]** **PKI Distribution**

- Click **OK**



## Importing the Root CA and Intermediate CA in the GPO



- With the named GPO created **right-click** the GPO and click **Edit…**

- Within our GPO, go to **Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies**

- Right-click **Trusted Root Certification Authorities** and click **Import…**

- In the pop-up wizard, continue until you reach **Browse…**

- **Open** the **Root** **.cer** file that is located in the Intermediate CA server's **inetpubwwwrootCertEnroll** folder (may be done with secure media or network share)

- Accept all wizard defaults and finish

- Repeat steps 3-6 with the **Intermediate Certification Authorities** folder and importing the **Intermediate CA .cer file**



## Enabling Auto-Enrollment Policy in the GPO



- Within our named GPO, go to **Computer Configuration > Policies > Windows Settings > Security Settings > Public Key Policies**

- Under Public Key Policies, double-click **Certificate Services Client - Auto-Enrollment**

- **Enable** the policy and **Check** the boxes for **Renew expired…** and **Update certificates…**

- Click **Ok**



## Creation of Windows Certificate Template



**In the Intermediate CA server**



- Open **Certification Authority (certsrv.msc)**

- Under our server, go to **Certificate Templates**

- Right click **Certificate Templates** and click **Manage**

- Right-click **Workstation Authentication** and click **Duplicate Template**

- In the **General** tab,

  - Name the template as necessary. I suggest **[INTERNAL-ORG] Windows Authentication**

  - Set the **Validity Period** and **Renewal Period** as necessary for workstation/server certificates (preferably **2 year** and **6 weeks** respectively)

  - Check **Publish certificate in Active Directory**

- In the **Compatibility** tab,

  - Set **Certification Authority** to **Windows Server 2008 R2** (must be compatible with oldest server)

  - Set **Certificate Recipient** to **Windows 7 / Server 2008 R2** (must be compatible with oldest server)

- In the **Subject Name** tab,

  - Select **Build from this Active Directory information**

  - Change **Subject name format** to **Common name**

  - Check the box for **DNS name**

- In the **Extensions** tab,

  - With **Application Policies** selected, click **Edit**

  - In the **Edit Application Policies Extension** window**,** click **Add**

  - In the **Add Application Policy** window, select **Server Authentication** and click **Ok**

- In the **Security** tab,

  - For the group **Domain Computers**, ensure the permissions, **Read**, **Enroll**, and **Autoenroll** are checked to **Allow**



**Note**: Ensure Domain Controllers are included in Domain Computers group, otherwise DCs to receive the same permissions



## Issuing the newly created Certificate Template



- In **Certificate Authority (certsrv.msc)**, under our server, right-click **Certificate Templates**, and click **New > Certificate Template to Issue**

- In the **Enable Certificate Templates**, select the newly created certificate template and click **Ok**



## Verification



- On any computer under the configured domain,

  - Run **gpupdate /force**

  - Run **certutil -pulse**

- Open **Local Machine Certificate Store (certlm.msc)**

- Check **Trusted Root Certification Authorities > Certificates** for **[INTERNAL-ORG]-ROOTCA-CA**

- Check **Intermediate Certification Authorities > Certificates** for a **[INTERNAL-ORG]-INTERCA-CA** certificate

- Check **Personal > Certificates** for a certificate issued by **[INTERNAL-ORG]-INTERCA-CA**.

