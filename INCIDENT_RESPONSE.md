# Incident Response Plan: TerraFlask Support Assistant

An incident is any answer or behaviour that is false, harmful, discriminatory, exposes personal data, or makes a commitment the company has not made.

## Severity levels

| Level | Examples | Response time |
|---|---|---|
| **1. Critical** | Personal data of one customer shown to another; a false promise repeated to many customers (for example "lifetime warranty"); offensive or discriminatory output | Contain within 1 hour |
| **2. High** | A wrong price, warranty term or safety-relevant care instruction given to a customer; a successful prompt injection | Contain within 1 business day |
| **3. Medium** | Repeated false refusals; wrong citations; poor answers in one language | Fix in the next scheduled update |
| **4. Low** | Awkward wording; a single unhelpful answer | Log and review weekly |

## How incidents are detected
- **Automated flags in the audit log:** blocked injections, failed number or citation checks, hand-overs.
- **Customer signals:** "Not helpful" ratings, "Talk to a team member" requests, e-mail complaints.
- **Weekly review** of a random sample of 50 conversations plus every flagged one.
- **Re-running the evaluation notebook** after any change.

## Response steps
1. **Contain.**
   - Level 1–2: switch the assistant to hand-over-only mode, so every question goes to the support team. This is a single configuration change.
   - Level 1: remove the affected document from the index.
2. **Assess.** Use the audit log to find every conversation affected, using the masked question, answer, sources and flags.
3. **Notify.**
   - **Customers** who received a false commitment are contacted with the correct information.
   - **Personal data breach:** if personal data was exposed, the data protection officer assesses it, and if it is likely to result in a risk to people, the supervisory authority is notified **within 72 hours** (GDPR Article 33). Affected people are informed without undue delay when the risk is high (Article 34).
4. **Fix.** Correct the document, prompt or safeguard. Add the failing case to the test set.
5. **Verify.** Re-run the full evaluation notebook; the new test must pass and no existing result may get worse.
6. **Restore.** Return to normal mode with sign-off from the support lead.
7. **Review.** Record cause, impact and fix within 5 business days, and update the risk register.

## Roles

| Role | Responsibility |
|---|---|
| Support lead | Owns the assistant; decides containment; signs off restoration |
| Document owner | Keeps each source document correct and dated |
| Data protection officer | Assesses personal-data incidents; handles GDPR notifications |
| Developer | Applies fixes; re-runs the evaluation |

## Record template
| Field | Content |
|---|---|
| Incident ID and date | |
| Severity | |
| What happened (with log IDs) | |
| Customers affected | |
| Containment action and time | |
| Root cause | |
| Fix and test case added | |
| Evaluation re-run result | |
| Sign-off | |
