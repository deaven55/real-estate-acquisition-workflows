# WF-087 Communication Chain Contract Test

## Purpose

WF-087 validates the communication-policy contract across the inbound/reply/runtime chain without sending an external message.

It covers:

- routine autonomous email
- routine autonomous SMS
- DNC/STOP suppression
- risk escalation
- human approval when autonomous mode is disabled
- missing inbound context
- provider receipt and idempotency requirements for autonomous candidates

## Chain

Routine autonomous candidate:

1. WF-083 inbound communication intake
2. WF-084 response-mode decision
3. WF-082 autonomous communication authorization/dispatch
4. WF-085 provider runtime adapter

Non-autonomous or blocked cases stop before provider dispatch.

## Safety

WF-087 is contract-only. It does not call the external provider and cannot send SMS or email.

Run controlled provider traffic only after WF-086 reports the requested channel ready and provider identity/configuration has been independently verified.

## Example

POST to the WF-087 webhook with:

{
  "scenario": "routine_sms"
}

Expected result:

- `passed: true`
- `observed_response_mode: autonomous_candidate`
- chain includes WF-083, WF-084, WF-082, WF-085
- provider receipt required
- deterministic idempotency required

Other scenarios:

- `routine_email`
- `routine_sms`
- `dnc`
- `risk_escalation`
- `approval_required`
- `missing_context`
