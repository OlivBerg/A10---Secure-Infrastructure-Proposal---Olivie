# A10: Secure Infrastructure Proposal (Olivie)

## Part A: Architecture Diagram

Identity providers connect into Microsoft Entra with SSO and MFA. Access splits into three paths: users receive app roles and groups, platform operations use least privilege RBAC, and services use managed identities so secrets are not embedded in code. Logs go to Log Analytics while Azure Policy and infrastructure as code supply prevention inputs. Microsoft Sentinel detects and manages alerts. The flow ends with containment and remediation such as revoking sessions and adjusting access.

![Architecture flowchart for Azure and Entra](./image.png)

| Step                        | Meaning                                                                                                                          |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Federation into Entra       | Schools and districts use their existing IdPs; traffic is federated into Entra with strong sign in including MFA where required. |
| User access                 | Application roles and group claims decide what tenant users can do inside the product.                                           |
| Platform operations         | People who run Azure use narrow role assignments instead of broad owner style access.                                            |
| Services                    | Workloads authenticate with managed identities instead of long lived passwords in repositories.                                  |
| Log Analytics               | Application, Azure control plane, and identity events land in one place for retention and search.                                |
| Policy and IaC              | Organization rules and reviewed templates reduce misconfiguration before and after deploy.                                       |
| Sentinel                    | Alerts and incidents are triaged in the security operations tool.                                                                |
| Containment and remediation | Sessions can be revoked, roles adjusted, and playbooks updated after an event.                                                   |

## Part B: Compliance Mapping Table

At least five requirements mapped to controls and evidence, spanning identity, visibility, and policy.

| Requirement                                          | Control (what we do)                                                                        | Evidence (what we show an auditor)                               |
| ---------------------------------------------------- | ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Identity: only authorized people can use the product | Federated SSO, MFA for sensitive accounts, least privilege roles, no shared admin passwords | Access policy summaries, sample sign in history                  |
| Identity: admins are identifiable and accountable    | Named admins, just in time elevation for rare tasks, audit trail on privileged actions      | Elevation logs, audit exports with user and action               |
| Visibility: logs are kept for review                 | Central logging with defined retention                                                      | Retention settings, archive policy                               |
| Visibility: suspicious activity is noticed           | Alerts on failed logins, role changes, unusual API or location patterns                     | Alert descriptions, example incident timeline                    |
| Policy: baseline security settings are enforced      | Organization wide guardrails; infrastructure defined in code and reviewed in CI             | Policy compliance report, failed build example from a bad change |
| Policy: environment matches approved design          | Regular checks that live Azure matches approved templates                                   | Drift or compliance scan output                                  |

## Part C: Incident Response Outline (one scenario)

Scenario: a stolen session or token is used to call admin APIs from an unusual location, suggesting account or token compromise.

| Phase       | Summary                                                                                                                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Detection   | Security monitoring flags a spike in admin API use combined with geo or risk signals from identity logs. On call is notified through the normal alerting path.                                                            |
| Evidence    | Pull application audit logs, identity sign in and token activity, gateway access logs if used, and the security tool incident record. Store copies under your retention rules if legal or contractual review is possible. |
| Containment | Invalidate the affected user sessions and tokens; temporarily restrict admin roles or APIs, block abusive IPs at the network edge and slow or stop sensitive jobs if needed.                                              |
| Remediation | Tighten conditional access for locations and MFA strength, rotate exposed secrets, remove exposed secrets, improve detection rules and runbooks, and notify the customer.                                                 |

## Brief justification of key design decisions and tradeoffs

1. One cloud security story on Azure: easier training, support, and audit evidence than stitching many unrelated vendors. The tradeoff is vendor concentration and cost as log volume grows.
2. Federation for education buyers: districts already have IdPs, so you integrate with them instead of forcing a new password store. Up front integration work supports trust and sales.
3. Preventive and detective controls: policies and pipelines catch mistakes before production; monitoring catches what still slips through. Weekly releases rely on automated checks instead of manual gatekeeping every time.
4. Identity for services: managed identities reduce credential leaks. The tradeoff is clear ownership of which service may access which data.

## Reflection Question

**What tradeoffs did you make to balance security, compliance, cost, and development velocity? What would you add with more time and budget?**

Tradeoffs: the design favors clarity and auditability with one stack, strong identity, and logging over minimal spend and maximum agility. Some risk is accepted and handled with detection and response rather than blocking every change by committee.

With more time and budget: deeper data governance for education data, formal compliance mapping, customer managed keys for the most sensitive tenants, regular simulated incidents and external penetration testing, and resilience testing so logging survives outages.
