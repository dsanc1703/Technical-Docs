[[View PDF Version]](<./Change-Management/PDF In-House Root-Intermediate CA Change Management.pdf>)

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

- Build and config the new offline Root CA (not domain-joined) [[SOP]](<./SOPs/CA Implementation/Step 1 Offline Root CA Creation>)
- Build and config the new online Intermediate CA (domain-joined) [[SOP]](<./SOPs/CA Implementation/Step 2 Online Intermediate CA Creation>)
  - Create Certificate Template for Windows devices [[SOP]](<./SOPs/CA Implementation/Step 3 Domain CA Distribution with Auto Enrollment>)
  - Create Certificate Template for Non-Windows devices [[SOP]](<./SOPs/CA Implementation/Step 4 Non-Windows Certificate Template>)
- Backup Group Policy of domains
- Notify of maintenance to any concerning parties

#### Implementing

- On plant domain controller [[SOP]](<./SOPs/CA Implementation/Step 3 Domain CA Distribution with Auto Enrollment>)
  - Create GPO for PKI Distribution
  - Import the Root CA certificate into created GPO
  - Enable Auto-Enrollment on GPO
  - Validate auto-issued certificate on a domain-joined device

- For each Non-Windows device, manually install signed certificate [[SOP]](<./SOPs/CA Implementation/Step 5 Non-Windows CA Distribution>)

#### Post-Implementing

- Monitor for any trust chain errors on devices
- Renew certficate authories and certification revocation list [[SOP]](<./SOPS/CA Implementation/PKI Lifecycle, Revoking, and Root Compromise>)
- Revoke child certificate and certificate authorities as necessary [[SOP]](<./SOPS/CA Implementation/PKI Lifecycle, Revoking, and Root Compromise>)

# Risk Assessment of Changes

Minimal-Low Risk to our operations

Possible risks:

- Incorrect certificate causing temporary trust issues (no trust to begin with)
- Misconfiguration of the CA trust chain causing HTTPS warnings (already prevalent)

These "risks" are all reversible through removal of certificate through Group Policy

# Rollback Plan

Should any issue arise:

- Remove implemnted CA certificates from the affected domain via GPO
- Revert to previous Group Policy version
- Force GPUpdate on impacted devices
