# Case 001 — Exchange Mailbox Permission Change

## Executive Summary

Wazuh generated an Office 365 alert for an Exchange Online `Add-MailboxPermission` operation. The event initially appeared security-relevant because it involved granting `FullAccess` to a mailbox. Investigation of the raw Microsoft 365 audit record showed that the target was a built-in `DiscoverySearchMailbox`, the principal receiving access was the `Discovery Management` role group, and the action was executed by the Exchange service identity `NT SERVICE\\MSExchangeAdminApiNetCore` from the IPv6 loopback address `::1` with `ExternalAccess` set to `false`.

Based on the available evidence, the activity was assessed as **likely benign Microsoft Exchange system/administrative activity** rather than malicious mailbox access.

**Final disposition:** True Positive — Benign / Authorized system activity

---

## Alert Summary

| Field | Observed value |
|---|---|
| Operation | `Add-MailboxPermission` |
| Wazuh rule ID | `91725` |
| Wazuh rule level | `10` |
| Subscription | `Audit.Exchange` |
| Workload | `Exchange` |
| Result | `True` |
| Target object | `DiscoverySearchMailbox{...}` |
| Principal granted access | `Discovery Management` |
| Access right | `FullAccess` |
| Actor | `NT SERVICE\\MSExchangeAdminApiNetCore (Microsoft.Exchange.AdminApi.NetCore)` |
| Client IP | `[::1]:11673` |
| External access | `false` |
| App pool | `MSExchangeAdminApiNetCore` |

Sensitive tenant-specific identifiers have been intentionally omitted from this public write-up.

---

## Why the Alert Was Investigated

Mailbox permission changes are high-value SOC telemetry because an attacker with administrative access can grant another account access to a victim mailbox without changing the victim's credentials.

`FullAccess` is especially important because it allows the granted principal to open and read mailbox contents. A successful `Add-MailboxPermission` operation against a sensitive user mailbox therefore warrants validation of:

1. who performed the change,
2. which mailbox was targeted,
3. which principal received access,
4. what permission was granted,
5. where the action originated,
6. whether related sign-in or administrative activity is suspicious.

The Wazuh alert was therefore treated as a legitimate investigation candidate rather than dismissed based only on severity.

---

## Evidence Review

### 1. Operation

```text
Add-MailboxPermission
```

This indicates that Exchange processed a mailbox permission change.

Conceptually, the audited operation is equivalent to:

```powershell
Add-MailboxPermission \
  -Identity "DiscoverySearchMailbox..." \
  -User "Discovery Management" \
  -AccessRights FullAccess
```

### 2. Target mailbox

```text
DiscoverySearchMailbox{...}
```

The affected object was not a normal end-user mailbox. It was a special Exchange discovery/system mailbox.

This significantly lowered the likelihood that the event represented unauthorized access to a user's business email.

### 3. Principal receiving access

```text
Discovery Management
```

The access was granted to the Exchange `Discovery Management` role group rather than an unfamiliar user or external account.

### 4. Permission granted

```text
FullAccess
```

This is a sensitive permission because it can allow the principal to open and read the mailbox.

The permission itself is security-relevant, even though the surrounding context in this case indicates expected service activity.

### 5. Actor

```text
NT SERVICE\MSExchangeAdminApiNetCore
```

The operation was executed by an Exchange administrative service identity rather than a named human administrator.

The event also identified the Exchange administrative API application pool:

```text
MSExchangeAdminApiNetCore
```

### 6. Source

```text
ClientIP: [::1]:11673
ExternalAccess: false
```

`::1` is the IPv6 loopback address. This indicates that the audited request originated internally on the service host rather than from a remote Internet address.

`ExternalAccess: false` provides additional context that the operation was not marked as externally initiated.

### 7. Result

```text
ResultStatus: True
```

The permission operation succeeded.

This means the alert reflects a real administrative action rather than a failed attempt.

---

## Analyst Reasoning

The operation deserved investigation because `Add-MailboxPermission` with `FullAccess` can be abused for unauthorized mailbox access.

However, the following indicators collectively point toward benign Exchange service activity:

- target is a built-in `DiscoverySearchMailbox`,
- recipient is the built-in `Discovery Management` role group,
- actor is the Exchange Admin API service identity,
- source address is IPv6 loopback (`::1`),
- `ExternalAccess` is `false`,
- result is a successful Exchange administrative operation,
- no evidence in the reviewed alert indicates an unfamiliar external user or remote source.

No single field is sufficient by itself. The disposition is based on the combination of actor, target, principal, source, and context.

---

## Classification

**True Positive — Benign / Authorized system activity**

The Wazuh rule correctly detected a real mailbox permission change. The alert was not a false positive because the underlying operation actually occurred.

It was classified as benign because the available evidence is consistent with expected Microsoft Exchange administrative/system behavior.

---

## What Would Make a Similar Alert Suspicious?

A future `Add-MailboxPermission` alert should receive higher priority if it contains indicators such as:

- target is an executive, finance, HR, administrator, or other sensitive user mailbox,
- recipient is a normal or newly created user account,
- `AccessRights` is `FullAccess`,
- actor is an unexpected administrator account,
- client IP is an unfamiliar external address,
- the action follows suspicious sign-in activity,
- multiple mailboxes receive permissions in a short period,
- permission is granted outside an approved change window,
- new inbox rules, forwarding rules, or delegate permissions appear shortly afterward.

Example suspicious pattern:

```text
Operation: Add-MailboxPermission
Identity: executive@company.example
User: unfamiliar-user@company.example
AccessRights: FullAccess
Actor: compromised-admin@company.example
ClientIP: unfamiliar external IP
ResultStatus: True
```

Such a pattern could indicate persistence, data theft, business email compromise, or abuse of administrative privileges.

---

## Recommended SOC Follow-Up

For similar alerts, an analyst should:

1. inspect the full Microsoft 365 audit event,
2. identify the mailbox target and granted principal,
3. verify the access right,
4. review the actor and source IP,
5. search Entra sign-in logs for the actor,
6. check for related Exchange permission changes,
7. review forwarding and inbox-rule activity,
8. confirm whether the change was approved,
9. escalate if the target, actor, or source is unexpected.

For benign recurring Microsoft system events, tuning should be considered carefully. Broadly suppressing all `Add-MailboxPermission` alerts would reduce visibility into genuinely dangerous permission changes.

A better future detection strategy is to lower priority only when known-safe conditions are present, while escalating mailbox permission changes involving human administrators, end-user mailboxes, unfamiliar principals, or external source IPs.

---

## Detection Engineering Opportunities

This case can be used as the basis for a higher-fidelity custom Wazuh rule.

Potential logic:

```text
IF Operation = Add-MailboxPermission
AND AccessRights contains FullAccess
AND actor is not a known Exchange service identity
THEN raise severity
```

Additional enrichment could consider:

- VIP mailbox lists,
- privileged administrator accounts,
- source-country anomalies,
- impossible-travel or risky sign-ins,
- newly created accounts,
- repeated permission grants,
- permission removal shortly after access,
- mailbox forwarding or inbox-rule creation.

---

## Investigation Workflow Used

```text
Wazuh alert
   |
   v
Search operation: Add-MailboxPermission
   |
   v
Expand Office 365 audit document
   |
   v
Identify actor / target / principal / rights
   |
   v
Review ClientIP and ExternalAccess
   |
   v
Assess human vs service activity
   |
   v
Determine risk and disposition
   |
   v
Document findings
```

---

## Lessons Learned

- A medium/high-severity alert is a starting point, not a verdict.
- Mailbox permission changes require context around actor, target, principal, and source.
- `FullAccess` is sensitive even when the underlying activity is legitimate.
- Microsoft service identities can generate security-relevant Exchange audit events.
- Loopback source addresses and built-in system mailboxes are useful contextual indicators.
- Correct SOC triage means distinguishing a valid detection from malicious activity rather than treating every alert as an incident.
- Wazuh's Office 365 integration provides enough raw event context to perform meaningful cloud-security investigations in a homelab.

---

## Portfolio Outcome

This case demonstrates hands-on experience with:

- Microsoft 365 audit telemetry,
- Exchange Online security events,
- Wazuh alert triage,
- mailbox permission analysis,
- cloud identity and administrative activity,
- false-positive/benign-positive classification,
- SOC investigation documentation,
- detection-engineering thinking.

This investigation was performed in a personal SOC homelab using Microsoft 365 Business Standard audit data ingested into Wazuh 4.14.5.
