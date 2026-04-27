# Change Summary

This change establishes a centralized Public Key Infrastructure across all plant networks by deploying a two-tier hierarchy (Offline Root CA and Online Intermediate CA) across each plant subnet. This allows devices and administrative workstations to trust HTTPS connections signed by our internal Certificate Authority, eliminating the constant ignorance of warnings should a legitimate MITM be in effect

# Purpose / Justification

Largely all our OT devices currently have expired certificates (from 2012), which causes constant certificate warnings and leaves HTTPS traffic open to man-in-the-middle attacks should a threat actor gain physical access to plant.

Benefits of an internal trusted Root CA:

- Trusted certificate validation
- No invalid certificate warnings when accessing https
- Reduced MITM risk on premises
- Establishes a consistent PKI foundation for future renewal

# Scope

#### In-Scope (what we're doing)

- Deployment of a new Windows Server VM/device to serve as the offline Root CA at the main office
- Deployment of a new Windows Server VM/device to serve as the online Intermediate CA at the main office
- Exporting the Root CA certificate to each plant's Domain Controller
- Installing the Root CA certificate into each domain's Trusted Root store via Group Policy

#### Out-of-Scope (what we're not doing)

- Removing/modifying existing device certificates
- Manual certificate installation on non-domain joined devices (Linux, RTUs, etc)

# Implementation Plan

#### Preparation

- Build and config the new offline Root CA (not domain-joined) [\[Refer to SOP\]]
- Build and config the new online Intermediate CA (domain-joined) [\[Refer to SOP\]]
- Have Root CA sign the Intermediate CA's Certificate Signing Request
- Export Root CA public certificate onto secure USB media or secure network share
- Backup Group Policy of domains
- Notify of maintenance to any concerning parties

#### Implementing

- On plant domain controller [\[Refer to SOP\]]
  - Import the Root CA certificate into: Group Policy: Computer Configuration -> Policies -> Windows Settings -> Security Settings -> Public Key Policies -> Trusted Root Certification Authorities
  - Deploy the updated GPO to all systems in the domain
  - Validate by connecting to test device that uses a certificate issued by the Intermediate CA
- Confirm trusted HTTPS connection

#### Post-Implementing

- Monitor for any trust chain errors on devices
- Document which domains received the new Root CA certificate

# Risk Assessment of Changes

Minimal-Low Risk to our operations

Possible risks:

- Incorrect certificate causing temporary trust issues (no trust to begin with)
- Misconfiguration of the CA trust chain causing HTTPS warnings (already prevalent)

These "risks" are all reversible through removal of certificate through Group Policy

# Rollback Plan

Should any issue arise:

- Remove the Root CA certificate from the affected domain via GPO
- Revert to previous Group Policy version
- Force GPUpdate on impacted devices