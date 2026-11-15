### ADR-0002: Enforce the schedule approval gate in the backend and exclude approval tools from the LLM

- Status: accepted
- Date: 2026-11-15
- Deciders: ME, VV
- Related: FR-02, FR-03, FR-05, DEC-2, R-02, R-06, GR-04

#### Context and problem
In a patient-facing medication assistant, an LLM could generate a flawed schedule or misinterpret timing rules. If the AI agent had direct write access to persist schedules into the PostgreSQL database, a hallucination could alter the patient's active regimen without their consent. Where and how should human approval be enforced?

#### Decision drivers
- Safety: Prevent unverified AI outputs from being persisted as active medication schedules
- FR-03: High-risk drug-drug interactions must be strictly blocked
- OWASP LLM06 (Excessive Agency): Limit autonomous write actions by the model

#### Considered options
1. Allow the Scheduling Agent to save schedules directly and let the patient edit/delete mistakes afterward
2. Enforce the approval check only in the frontend Web UI before calling the save endpoint
3. Enforce a state machine (`PENDING_APPROVAL` -> `APPROVED` -> `SAVED`) in the backend/MCP layer and keep the approval action exclusively triggered by the UI (never exposed as an LLM tool)

#### Decision
Chosen option 3, because backend enforcement guarantees that neither a prompt injection nor an agent loop can bypass human review, and `BLOCKED` proposals (due to HIGH interactions) can never transition to `APPROVED`.

#### Consequences
- Good: Strong safety guarantee ("AI proposes, human disposes") and clear audit trail of approvals.
- Bad: Adds extra state management in the backend and requires an additional user click before reminders become active.

#### Confirmation
Tested in MCP Inspector and Lab 6 by attempting to call `save_schedule` on `PENDING_APPROVAL` and `BLOCKED` proposals (must return `REFUSED`).
