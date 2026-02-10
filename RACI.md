## RACI Matrix: Application Operations & Maintenance

| Task/Activity                                  | Business | Internal Engineering | Viden | SLA (viden)                                                           |
| ---------------------------------------------- | -------- | -------------------- | ----- | --------------------------------------------------------------------- |
| **Operational Tasks**                          |
| Application monitoring & alerting              | I        | R/A                  | C/I   | -                                                                     |
| Incident response (L1/L2)                      | I        | R/A                  | C     | -                                                                     |
| Incident escalation (L3)                       | I        | A                    | R     | 4h response, 8h resolution target during business hours               |
| Pre-update validation (limits, scope, dry-run) | I        | C                    | R/A   | Mandatory before each production release                              |
| Production deployments                         | A        | R                    | C     | -                                                                     |
| Infrastructure management                      | I        | R/A                  | C     | -                                                                     |
| User access management                         | A        | R                    | I     | -                                                                     |
| Backup & recovery execution                    | I        | R/A                  | C     | -                                                                     |
| **Development & Enhancement**                  |
| New feature requirements                       | R/A      | C                    | I     | -                                                                     |
| New feature development                        | A        | C                    | R     | Per project agreement|
| Bug fixes (critical)                           | A        | I                    | R     | 24-hour fix deployment                                                |
| Bug fixes (high priority)                      | A        | I                    | R     | 3 business days                                                       |
| Bug fixes (medium/low)                         | A        | I                    | R     | 10 business days                                                      |
| Code reviews for new features                  | I        | C                    | R/A   | -                                                                     |
| **Maintenance & Support**                      |
| Security patches                               | I        | A                    | R     | 5 business days (critical)<br>15 business days (non-critical)         |
| Dependency updates                             | I        | C/A                  | R     | Quarterly                                                             |
| Performance optimization                       | I        | A                    | R     | Per issue severity                                                    |
| Technical debt resolution                      | C        | A                    | R     | Per agreement                                                         |
| **Documentation & Knowledge**                  |
| Technical documentation                        | I        | C                    | R/A   | Delivered with each release                                           |
| Runbook maintenance                            | I        | R/A                  | C     | -                                                                     |
| Knowledge transfer sessions                    | A        | C                    | R     | Quarterly or as needed                                                |
| Architecture decision records                  | C        | A                    | R     | With major changes                                                    |
| **Quality & Compliance**                       |
| Code quality standards                         | C        | A                    | R     | -                                                                     |
| Testing (unit/integration)                     | I        | C                    | R/A   | Main execution flows covered by integration tests. (behavior-focused) |
| UAT coordination                               | R        | C                    | A     | -                                                                     |
| Production issue RCA                           | I        | A                    | R     | 5 business days post-resolution                                       |
| Compliance audits                              | R/A      | C                    | I     | -                                                                     |

### RACI Legend

- **R** = Responsible (does the work)
- **A** = Accountable (final decision-maker)
- **C** = Consulted (provides input)
- **I** = Informed (kept updated)

### Additional SLA Notes

**Business hours:** Mon–Fri 09:00–17:00 CET

**Incident Priority & Response Times (Viden SLA):**
- **P1 (Critical / client-impacting):** 4-hour response, 8-hour resolution target
- **P2 (High priority):** Next business day response
- **P3 (Standard):** 3 business day response

**Support Levels (L1/L2/L3):**
- **L1 (Internal Engineering):** Initial triage and coordination. Acknowledge incident, check dashboards/logs, validate impact, gather context, and apply safe immediate actions (pause/stop further changes, disable the scheduled run, retry with backoff if safe) per runbook.
- **L2 (Internal Engineering):** Deeper technical investigation. Reproduce issues, isolate root cause within client infrastructure/code/config, implement and deploy fixes that do not require external vendor changes, coordinate client communications, and support remediation by providing logs, run context, and operational tooling (Google Ads changes are handled by Viden/Business as applicable).
- **L3 (Viden / External Company):** Escalation for complex issues requiring vendor expertise or changes (e.g., ad update logic correctness, domain-specific behavior, Google Ads API schema/policy changes, quota/throttling handling strategy). Owns the L3 response/mitigation targets above.

**Scheduled Run Cadence:**
- Scheduled job runs weekly on Monday (and/or other days as needed, datetime TBD per deployment)

**P1 Incident Definition (Client Impacting):**
- Failed or missed weekly update
- Incorrect changes applied to target Google Ads account

**P1 Targets:**
- **Response:** 4 business hours
- **Mitigation:** 8 business hours (aligned with L3 SLA)
- **Note:** P1 incidents involving Google Ads account correctness escalate directly to L3 (Viden/Business), skipping L2.

**Third-party dependency (Google Ads API):**
- Google Ads API availability, quotas, schema changes, and policy enforcement
- SLA does not apply to failures caused by Google Ads API outages, throttling, or breaking changes.
- Viden is responsible for monitoring Google Ads API changes and maximizing quota utilization (without triggering restrictions)

**Client-side responsibilities (when running on client infrastructure):**
- OAuth credentials storage and rotation
- Least-privilege access to Google Ads accounts

**Accountability boundaries:**
- Viden is responsible for technical correctness of updates.
- Internal Engineering is responsible for infrastructure, deployments, and operational tooling, but is not responsible for the semantics/correctness of Google Ads account changes or for tracking Google Ads policy/schema changes.
- Business remains responsible for strategic advertising decisions and performance outcomes.

**Escalation Path:**
1. Internal Engineering (L1/L2) → External Company (Viden) (L3)
2. Google Ads–specific P1 incidents → directly to External Company (Viden) (L3), skipping L2
3. SLA breach: External Company (Viden) → Internal Engineering → Business Stakeholder

**Communication Requirements:**
- Monthly status meetings with all parties
- Incident reports within 24 hours of resolution
- Quarterly roadmap reviews
