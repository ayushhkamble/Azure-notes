
## 1. Multi-Factor Authentication (MFA)

### What is MFA?
<cite index="1-1">Multifactor authentication is a process in which users are prompted during the sign-in process for an additional form of identification, such as a code on their cellphone or a fingerprint scan.</cite> <cite index="1-1">Microsoft Entra multifactor authentication works by requiring two or more of the following authentication methods: Something you know (typically a password), Something you have (a trusted device that's not easily duplicated, like a phone or hardware key), and Something you are (biometrics like a fingerprint or face scan).</cite>

### Why It Matters
<cite index="1-1">If you only use a password to authenticate a user, it leaves an insecure vector for attack — if the password is weak or has been exposed elsewhere, an attacker could be using it to gain access. When you require a second form of authentication, security is increased because this additional factor isn't something that's easy for an attacker to obtain or duplicate.</cite>

### Three Ways to Enable MFA
| Method | How It Works | Requires |
|---|---|---|
| **Security Defaults** | <cite index="3-1">A simple, tenant-wide toggle available in all Microsoft 365 organizations via Microsoft Entra ID Free — turned on by default for tenants created after October 2019</cite> | Entra ID Free (no extra license) |
| **Conditional Access Policies** | <cite index="1-1">More granular controls — define events or applications that require MFA, allowing regular sign-in on the corporate network but prompting for verification when remote or on a personal device</cite> | Entra ID P1 or P2 license |
| **Azure Policy self-enforcement** | <cite index="8-1">Built-in policy definitions that can self-enforce MFA, supporting both Audit (reports noncompliance) and Deny (blocks noncompliant requests) effects</cite> | Ties directly into Section 4 (Azure Policy) |

### ⚠️ Important Update: Mandatory MFA for Azure Sign-In
<cite index="8-1">Microsoft has been rolling out mandatory multifactor authentication (MFA) enforcement for Azure, Microsoft 365, and other admin portals.</cite> <cite index="8-1">There's no change for users if your organization already enforces MFA for them, or if they sign in with stronger methods like passwordless or passkey (FIDO2).</cite>

> 🎯 **Teaching point:** Because of this rollout, treat MFA as **assumed baseline security**, not an optional add-on — new tenants and Azure sign-ins increasingly require it by default rather than by admin choice.

### SSPR Registration Overlap
<cite index="1-1">When users register themselves for Microsoft Entra multifactor authentication, they can also register for self-service password reset in one step</cite> — this ties directly into **Section 3**.

📘 **Official Docs:**
- [Microsoft Entra multifactor authentication overview – Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-mfa-howitworks)
- [Plan for mandatory Microsoft Entra multifactor authentication (MFA) – Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-mandatory-multifactor-authentication)

### 🧪 Practice Lab
1. In the Portal, go to **Microsoft Entra ID → Overview → Properties** and scroll to **Security defaults** — note whether it's currently on or off for your tenant.
2. Go to **Entra ID → Conditional Access → Overview** → **+ Create new policy** → walk through configuring a basic "require MFA for all users" policy (don't need to enable it on a live tenant).
3. Discuss: why might a company choose Conditional Access policies over Security Defaults, even though Conditional Access requires a paid license?

---

## 2. Microsoft Entra ID Protection

### What is Identity Protection?
<cite index="17-1">Microsoft Entra ID Protection helps organizations detect, investigate, and remediate identity-based risks. These risks can be fed into tools like Conditional Access to make access decisions, or sent to a security information and event management (SIEM) tool for further investigation and correlation.</cite>

### The Three-Step Cycle: Detect, Investigate, Remediate
```
1. DETECT   →  Continuous analysis of trillions of daily signals
2. INVESTIGATE  →  Risk reports flag risky users/sign-ins
3. REMEDIATE   →  Conditional Access blocks, requires MFA, or forces password reset
```

### Detecting Risk
<cite index="17-1">Detections come from analysis of trillions of signals each day from Active Directory, Microsoft Accounts, and Xbox gaming — this broad range of signals helps ID Protection detect risky behaviors like anonymous IP address usage, password spray attacks, and leaked credentials.</cite>

<cite index="17-1">During each sign-in, ID Protection runs all real-time sign-in detections, generating a sign-in session risk level that indicates how likely the sign-in is compromised — policies are then applied based on this risk level.</cite>

### Two Types of Risk
| Risk Type | Meaning |
|---|---|
| **Sign-in risk** | <cite index="16-1">The probability that a given authentication request isn't authorized by the identity owner — can be calculated in real-time or offline</cite> |
| **User risk** | <cite index="16-1">The probability that a given identity or account is compromised — calculated offline using Microsoft's internal and external threat intelligence sources</cite> |

### The Three Key Reports
<cite index="17-1">ID Protection provides three key reports for administrators to investigate risks: Risk detections (each individual risk detected), Risky sign-ins (reported when one or more risk detections are tied to that sign-in), and Risky users (reported when a user has one or more risky sign-ins, or one or more risk detections).</cite>

### Location and Trusted Networks
<cite index="14-1">Location in risk detections is determined using IP address lookup — sign-ins from trusted named locations improve the accuracy of ID Protection's risk calculation, lowering a user's sign-in risk when they authenticate from a location marked as trusted.</cite>

### High-Priority Detection Example — Leaked Credentials
<cite index="15-1">Leaked credentials detections are always high risk because they represent confirmed credential exposure — when this detection fires, investigate right away, checking whether the leaked credential was used for unauthorized access and whether the password has already been changed.</cite> <cite index="15-1">A cloud-based password reset triggered by a Conditional Access policy fully remediates the user risk for this detection.</cite>

### Actions Admins Can Take on a Detection
<cite index="13-1">Dismiss sign-in risk is used for a benign true positive — the risk detected is real, but not malicious, such as one from a known penetration test or an approved application. Similar sign-ins should continue being evaluated for risk going forward.</cite>

### Licensing
<cite index="12-1">Microsoft Entra ID Free, P1, or P2 licenses provide access, but Microsoft Entra ID P2 licenses are required to view a comprehensive list of security recommendations and select the recommended action links.</cite>

📘 **Official Docs:**
- [What is Microsoft Entra ID Protection? – Microsoft Learn](https://learn.microsoft.com/en-us/entra/id-protection/overview-identity-protection)
- [Risk detection types and levels – Microsoft Learn](https://learn.microsoft.com/en-us/entra/id-protection/concept-risk-detection-types)

### 🧪 Practice Lab
1. In the Portal, search **"Identity Protection"** → **Dashboard** — review the summary metrics (this may show "no risks detected" in a fresh training tenant, which is expected).
2. Explore the three reports: **Risk detections**, **Risky sign-ins**, and **Risky users** — note the filter/column options on each.
3. Discuss: why is a "leaked credentials" detection always treated as high risk, while other detections (like sign-in from a new but plausible location) might only be medium or low risk?

---

## 3. Self-Service Password Reset (SSPR)

### What is SSPR?
<cite index="19-1">Microsoft Entra self-service password reset (SSPR) gives users the ability to change or reset their password, with no administrator or help desk involvement. If a user's account is locked or they forget their password, they can follow prompts to unblock themselves and get back to work — this reduces help desk calls and loss of productivity.</cite>

### The Four SSPR Features
<cite index="18-1">Features that make up SSPR include password change, reset, unlock, and writeback to an on-premises directory.</cite>

### How SSPR Verification Works
<cite index="19-1">After the SSPR portal is displayed, the user enters a user ID and passes a captcha. Microsoft Entra ID then checks: that the user has SSPR enabled; that the user has the right authentication methods defined on their account in accordance with administrator policy — if the policy requires only one method, checking that at least one is configured, or if it requires two methods (a "two-gate" policy), checking both are set up.</cite>

### Password Writeback — Bridging Cloud and On-Premises
<cite index="20-1">Most companies also have an on-premises Active Directory Domain Services (AD DS) environment for users. Password writeback allows password changes in the cloud to be written back to an on-premises directory in real time by using either Microsoft Entra Connect or Microsoft Entra Connect Cloud Sync.</cite>

> 🎯 **Teaching point:** This directly connects to **Day 14's Hybrid Identity** lesson — without writeback, a user resetting their password via SSPR would end up with a cloud-only password that's out of sync with their on-premises AD password, breaking single sign-on to on-prem resources.

<cite index="20-1">If the writeback service is down, the user is informed that their password can't be reset right now — writeback requires the service to be up and running.</cite>

### Password Policy Enforcement
<cite index="23-1">In Microsoft Entra ID, there's a password policy that defines settings like password complexity, length, or age. When SSPR is used to change or reset a password, the policy is checked — if the password doesn't meet policy requirements, the user is prompted to try again.</cite>

### Administrator Accounts and SSPR
<cite index="23-1">Azure administrators have some restrictions on using SSPR that are different from regular user accounts — you can disable the use of SSPR for administrator accounts by setting the AllowedToUseSspr property on the tenant authorization policy to false. Policy changes to enable or disable SSPR for administrator accounts can take up to 60 minutes to take effect.</cite>

### Enabling SSPR — Scope Options
<cite index="25-1">You can enable SSPR for all users, no users, or for selected groups of users — only one Microsoft Entra group can currently be enabled for SSPR using the Microsoft Entra admin center, though nested groups are supported as part of a wider deployment.</cite>

### Why SSPR Matters — The Business Case
<cite index="24-1">Manage cost: SSPR reduces IT support costs by enabling users to reset passwords on their own, reducing the cost of time lost due to lost passwords and lockouts. Intuitive user experience: a one-time user registration process lets users reset passwords and unblock accounts on-demand from any device or location.</cite>

### Licensing
<cite index="18-1">Basic SSPR features are available in Microsoft 365 Business Standard or higher and all Microsoft Entra ID P1 or P2 SKUs at no additional cost.</cite>

📘 **Official Docs:**
- [Self-service password reset deep dive – Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-sspr-howitworks)
- [On-premises password writeback with SSPR – Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-sspr-writeback)
