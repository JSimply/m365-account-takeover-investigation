# Microsoft 365 Account Takeover Investigation

## Overview

This project documents a simulated Microsoft 365 account takeover investigation.

The goal is to demonstrate a realistic cybersecurity investigation workflow using identity logs, KQL, timeline analysis, MITRE ATT&CK mapping, and remediation recommendations.

This project uses synthetic data only and does not contain any real customer, employer, or proprietary information.

## Scenario

A Microsoft 365 user account generated suspicious authentication activity from multiple geographic locations in a short period of time.

Additional activity suggested that the attacker may have attempted to establish persistence and access mailbox data.

The investigation focuses on determining:

- Whether the account was compromised
- How the attacker gained access
- What actions were taken after authentication
- Whether persistence was established
- What remediation steps are required

## Tools and Technologies

- Microsoft Entra ID
- Microsoft Sentinel
- KQL
- Microsoft 365 Audit Logs
- MITRE ATT&CK
- Incident timeline analysis

## Investigation Phases

1. Initial alert review
2. Authentication analysis
3. Timeline reconstruction
4. Mailbox and OAuth activity review
5. MITRE ATT&CK mapping
6. Findings and conclusion
7. Remediation recommendations

## Project Status

🚧 In progress