# IAM Engineering Portfolio

A portfolio of Identity & Access Management engineering case studies focused on designing, implementing, validating, and troubleshooting enterprise identity controls.

The projects in this repository use Microsoft Entra ID as the initial identity platform and progressively cover identity lifecycle management, access control, Zero Trust, privileged identity, enterprise authentication, automation, and identity security operations.

The emphasis is not only on configuration, but on the engineering decisions behind each control: **why it exists, how it should behave, how it is validated, what can fail, and how the design can scale.**

## Engineering Focus

- Identity lifecycle management and Joiner-Mover-Leaver (JML) controls
- Group-based authorization and Role-Based Access Control (RBAC)
- Least-privilege administration
- Conditional Access and Zero Trust
- Privileged identity and administrative access
- Enterprise application authentication and SSO
- Microsoft Graph and PowerShell automation
- Identity monitoring and incident investigation
- Identity governance and enterprise architecture

## IAM Engineering Case Studies

### 01 — Enterprise Entra ID Identity Lifecycle

Implemented a workforce identity lifecycle and group-based authorization model for the fictional Northstar Technologies environment.

The implementation covers workforce identity provisioning, departmental security groups, a Finance-to-Sales mover workflow, leaver access termination, session revocation, delegated User Administrator access, least privilege, and final-state validation.

**Technical focus:** Microsoft Entra ID · JML · Security Groups · Delegated Administration · Least Privilege · Access Validation

[View the Enterprise Entra ID Identity Lifecycle case study](./01-entra-identity-lifecycle/)

---

### 02 — Conditional Access & Zero Trust

**Planned:** Identity-driven access controls, MFA strategy, privileged access protection, exclusions, emergency access, and policy testing.

### 03 — RBAC & Privileged Identity

**Planned:** Enterprise role design, administrative privilege, least privilege, and privileged-access controls where supported.

### 04 — Enterprise Application Authentication

**Planned:** Application integration, SSO, authentication protocols, assignments, claims, and authentication-flow troubleshooting.

### 05 — IAM Automation with Microsoft Graph / PowerShell

**Planned:** Identity administration automation, group management, lifecycle operations, reporting, and access analysis.

### 06 — IAM Monitoring & Incident Investigation

**Planned:** Sign-in analysis, audit activity, authentication failures, administrative changes, and identity incident investigation.

### 07 — Enterprise Identity Architecture Capstone

**Planned:** Integrated identity architecture combining lifecycle, authorization, authentication, privileged access, monitoring, and governance.

## Portfolio Approach

Each case study is documented around the same engineering lifecycle:

`Business Requirement → Architecture → Implementation → Security Reasoning → Validation → Troubleshooting → Operational Considerations`

Configuration evidence is sanitized before publication. Credentials, secrets, access tokens, tenant identifiers, subscription identifiers, and personal information are excluded from published artifacts.





