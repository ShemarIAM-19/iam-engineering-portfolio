# Enterprise Entra ID Identity Lifecycle
## Overview

This case study implements a workforce identity lifecycle and access model in Microsoft Entra ID for a fictional organization, Northstar Technologies.

The design focuses on centralized identity administration, group-based access, Joiner-Mover-Leaver (JML) lifecycle controls, delegated administration, and least-privilege principles.

Rather than assigning access directly to individual identities, departmental security groups establish an authorization layer between workforce identities and downstream resources.

The implementation also demonstrates how identity attributes, group memberships, and administrative roles operate as separate control layers that must remain aligned throughout the identity lifecycle.

## Business Scenario

Northstar Technologies represents an organization of approximately 500 employees distributed across Finance, Human Resources, Engineering, Sales, and IT.

The organization requires an identity model that can:

- Centrally manage workforce identities.
- Provide access according to business function.
- Reduce reliance on direct user-to-resource assignments.
- Support employee transfers without accumulating obsolete access.
- Rapidly restrict access when an employee leaves the organization.
- Separate standard business access from administrative privilege.
- Apply least privilege to delegated administrative responsibilities.
- Provide a structure that can evolve toward greater automation and governance as the organization scales.

## Identity Architecture

The core authorization model is:

**Workforce Identity → Department Security Group → Downstream Resource Access**

Administrative privilege is handled separately:

**IT Support Identity → Delegated Microsoft Entra Role**

This separation is intentional. Department group membership represents business access, while Microsoft Entra administrative roles provide authority to manage identity infrastructure.

Keeping these authorization layers separate reduces unnecessary privilege and provides clearer lifecycle and auditing boundaries.

## Environment

- **Identity Platform:** Microsoft Entra ID
- **Organization:** Northstar Technologies (fictional)
- **Identity Model:** Cloud-based workforce identities
- **Group Model:** Assigned security groups
- **Departments:** Finance, HR, Engineering, Sales, IT
- **Administrative Model:** Delegated Microsoft Entra directory role
- **Lifecycle Model:** Joiner-Mover-Leaver (JML)

> **Scope note:** This is an isolated IAM engineering environment created for implementation, validation, and documentation. Northstar Technologies is fictional, and the configuration documented here does not represent a production deployment.
> ## Identity and Group Design

The workforce model uses fictional identities representing common business functions across Northstar Technologies.

| Identity | Business Function | Department | Department Group |
|---|---|---|---|
| Maya Chen | Sales Representative | Sales | `SG-Department-Sales` |
| Daniel Brooks | HR Specialist | HR | None — account disabled during leaver workflow |
| Priya Shah | Software Engineer | Engineering | `SG-Department-Engineering` |
| Marcus Reed | Sales Representative | Sales | `SG-Department-Sales` |
| Elena Torres | IT Support Specialist | IT | `SG-Department-IT` |

> Maya Chen originally represented a Finance employee and was transitioned to Sales as part of the mover lifecycle scenario. Daniel Brooks was originally assigned to the HR group before the leaver workflow was executed.

### Department Security Groups

The following assigned security groups establish the departmental access structure:

- `SG-Department-Finance`
- `SG-Department-HR`
- `SG-Department-Engineering`
- `SG-Department-Sales`
- `SG-Department-IT`

The groups use an `SG-Department-<Department>` naming convention to make their purpose identifiable and provide a consistent structure for administration and auditing.

### Why Group-Based Access?

Directly assigning resource access to individual users creates administrative overhead and makes entitlement reviews more difficult as an organization grows.

The model instead separates identities from downstream access:

`Identity → Security Group → Resource`

A user's group membership therefore becomes the primary mechanism for representing departmental entitlement.

For example, a Sales resource could be assigned to `SG-Department-Sales` rather than separately assigning every Sales employee.

When personnel change roles, access can then be managed by changing group membership rather than locating and modifying multiple individual resource assignments.

> **IAM Engineering Note:** The department groups in this implementation establish the entitlement layer. Downstream production applications and resources were not configured as part of this case study.

## Joiner-Mover-Leaver Lifecycle

Identity lifecycle controls are designed around three primary events:

**Joiner:** Provision the identity, establish appropriate business attributes, and assign required group membership.

**Mover:** Revoke obsolete group membership, update identity attributes, provision the new entitlement, and validate the resulting access state.

**Leaver:** Block authentication, revoke active sessions, remove business entitlements, and retain the identity object when temporary retention is required for audit, recovery, or organizational processes.
## Mover Workflow — Finance to Sales

A mover event was simulated for Maya Chen to represent an employee transferring from Finance to Sales.

### Initial State

- Department: Finance
- Department group: `SG-Department-Finance`
- Account state: Enabled

### Lifecycle Actions

1. Removed Maya from `SG-Department-Finance`.
2. Updated the Department attribute from Finance to Sales.
3. Updated the job title to Sales Representative.
4. Added Maya to `SG-Department-Sales`.
5. Validated the resulting identity and group membership state.

### Result

Maya's final state reflected the new Sales business function:

- Department: Sales
- Department group: `SG-Department-Sales`
- Previous Finance membership: Removed
- Account state: Enabled

### Security Reasoning

A department transfer is not simply a provisioning event. It also requires revocation of access associated with the employee's previous responsibilities.

If Sales access were granted without removing Finance access, the identity could accumulate permissions that are no longer justified by the employee's current role.

This creates **privilege accumulation**, sometimes referred to as access creep.

The mover workflow therefore follows the pattern:

`Revoke obsolete entitlement → Update identity attributes → Provision new entitlement → Validate final state`

### Troubleshooting Perspective

If a transferred employee retained both Finance and Sales memberships, an IAM Engineer should compare the user's current business attributes against existing entitlements.

The investigation would include:

- Confirming the authoritative department and job-function attributes.
- Reviewing current group memberships.
- Identifying memberships associated with the previous role.
- Removing obsolete entitlements.
- Validating the final access state.

This scenario demonstrates why changing an identity attribute alone does not automatically correct access when assigned group membership is being used.

---

## Leaver Workflow — Access Termination

A leaver event was simulated using Daniel Brooks, representing an HR employee leaving the organization.

### Initial State

- Department: HR
- Department group: `SG-Department-HR`
- Account state: Enabled

### Lifecycle Actions

1. Disabled the user account to prevent new authentication.
2. Revoked active sessions.
3. Removed membership from `SG-Department-HR`.
4. Retained the identity object rather than immediately deleting it.
5. Validated that the account was disabled and departmental membership had been removed.

### Result

Daniel's final identity state was:

- Account state: Disabled
- HR department membership: Removed
- Identity object: Retained

### Security Reasoning

Offboarding requires more than removing a user from a security group.

Disabling the account prevents new authentication attempts, while session revocation addresses existing authenticated sessions. Removing business entitlements then reduces residual authorization associated with the identity.

The workflow follows the pattern:

`Disable authentication → Revoke sessions → Remove entitlements → Validate → Retain/Delete according to policy`

Immediate deletion was intentionally avoided in this implementation. Organizations may require temporary identity retention for audit, recovery, investigation, legal, or HR processes before final deletion occurs.

### Troubleshooting Perspective

If a leaver continued to have access after the account was disabled, investigation should not stop at the account's enabled/disabled state.

An IAM Engineer should also examine:

- Existing authenticated sessions.
- Group memberships and application assignments.
- Administrative role assignments.
- Other access paths associated with the identity.
- Relevant sign-in and audit activity where available.

This illustrates the difference between **authentication control** and **authorization cleanup** during offboarding.
## Delegated Administration and Least Privilege

Administrative access was separated from standard departmental access.

Elena Torres represents an IT Support identity. Her standard business access is represented through membership in:

`SG-Department-IT`

Administrative responsibility is represented separately through the Microsoft Entra:

`User Administrator`

role.

### Design Decision

Elena was delegated the User Administrator role rather than Global Administrator.

This follows the principle of least privilege: an administrator should receive only the permissions required to perform the intended administrative function.

The authorization model therefore contains two distinct layers:

`Elena Torres → SG-Department-IT → Business/Department Entitlement`

`Elena Torres → User Administrator → Delegated Directory Administration`

Membership in an IT security group does not automatically provide Microsoft Entra administrative authority. Likewise, assigning a directory role does not replace the business-access model represented by security groups.

### Why Not Global Administrator?

Global Administrator provides extensive control across a Microsoft Entra tenant. Assigning that level of privilege when a narrower administrative role can satisfy the requirement unnecessarily increases the potential impact of account compromise, administrative error, or misuse.

For this scenario, User Administrator provides a more appropriate demonstration of delegated identity administration.

> **IAM Engineering Note:** The tenant creator retains Global Administrator privileges for administration of this isolated lab environment. This should not be interpreted as a recommended production privileged-access model. Production environments should minimize standing high-privilege access and apply stronger privileged-access controls where available.

### Troubleshooting Perspective

When troubleshooting an administrator who cannot perform an expected task, group membership alone should not be treated as evidence of administrative authority.

An IAM Engineer should verify:

- Which Microsoft Entra directory role is assigned.
- Whether the role contains the permissions required for the operation.
- Whether the assignment applies to the correct identity.
- Whether the target object or operation has additional privilege requirements.
- Whether the role assignment is active and effective.

This distinction is important when troubleshooting authorization failures: **business group membership and directory administrative roles serve different purposes.**

---

## Testing and Validation

The implementation was validated against the expected lifecycle and authorization state.

| Identity | Expected State | Validated State | Result |
|---|---|---|---|
| Maya Chen | Enabled, Sales department, Sales group only | Enabled, `SG-Department-Sales`; previous Finance membership removed | Pass |
| Daniel Brooks | Disabled, HR entitlement removed | Account disabled; `SG-Department-HR` membership removed | Pass |
| Priya Shah | Enabled, Engineering entitlement | `SG-Department-Engineering` membership present | Pass |
| Marcus Reed | Enabled, Sales entitlement | `SG-Department-Sales` membership present | Pass |
| Elena Torres | Enabled, IT entitlement and delegated administration | `SG-Department-IT` membership and User Administrator role present | Pass |

### Validation Strategy

Validation was performed from both the identity and authorization perspectives rather than assuming that a successful configuration action produced the intended final state.

For lifecycle events, the final identity attributes and group memberships were reviewed after the changes were applied.

For delegated administration, Elena's directory role assignment was validated separately from her departmental group membership.

This provides an important operational principle:

`Requested Change ≠ Verified Access State`

An IAM change should be considered complete only after the resulting identity and entitlement state has been validated.
