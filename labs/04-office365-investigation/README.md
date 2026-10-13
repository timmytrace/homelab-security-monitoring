# Lab 04 — Office 365 Alert Investigation

## Objective

Practice the SOC investigation workflow using real Microsoft 365 alerts ingested into Wazuh.

The first case will focus on mailbox permission changes such as:

```text
Add-MailboxPermission
```

## Analyst Workflow

```text
Alert
  -> Understand the rule
  -> Inspect the raw event
  -> Identify user / host / IP
  -> Build a timeline
  -> Search related activity
  -> Determine scope
  -> Classify the event
  -> Recommend action
  -> Document findings
```

## Investigation Questions

For each alert, determine:

- What happened?
- Who initiated it?
- Which account or resource was affected?
- What permission or action was performed?
- What client IP was involved?
- Was the actor a human administrator, user, or Microsoft service account?
- Was the activity expected?
- What related events happened before and after it?
- Is the event malicious, benign, a false positive, or inconclusive?

## Useful Wazuh Filters

```text
rule.groups:office365
```

Search by operation when needed:

```text
Add-MailboxPermission
```

For broad investigation, remove restrictive severity filters such as:

```text
rule.level: 7 to 11
```

when they hide lower-severity audit activity.

## Evidence to Capture

Sanitized screenshots should include:

1. Office 365 dashboard overview
2. Expanded Wazuh event
3. Relevant event fields
4. Timeline of related activity
5. Final classification

Never capture passwords, tokens, client secrets, private keys, Wi-Fi credentials, or sensitive personal information.

## Case Record Template

### Alert

```text
Operation:
Wazuh rule ID:
Wazuh rule level:
Timestamp:
User:
Client IP:
Workload:
Result:
Affected object:
```

### Initial Assessment

Describe why the event was selected for investigation.

### Timeline

| Time | Event | User | Source | Notes |
|---|---|---|---|---|
| | | | | |

### Analysis

Document observations from the raw Microsoft 365 event and any correlated Wazuh events.

### Classification

Choose one:

- True Positive — Malicious
- True Positive — Benign / Authorized
- False Positive
- Needs Further Investigation

### Root Cause / Explanation

Explain why the event occurred based on the evidence.

### Recommended Action

Document containment, remediation, tuning, or monitoring recommendations.

### Lessons Learned

Record what this investigation taught about Wazuh, Microsoft 365 auditing, and SOC analysis.

## Next Detection Engineering Exercise

After the investigation is complete, create a custom Wazuh detection for a meaningful Office 365 condition, for example a high-risk mailbox permission change.
