# RACB Voice Platform (ANA)

**Evidence snapshot:** 2026-10-10  
**Maturity:** In Development  
**Source:** Private implementation repository; public claims limited to documented implementation and validation.

## Purpose

A governed operational foundation for an AI receptionist, connecting inbound telephony, structured lead capture, human escalation and future CRM/scheduling integration.

## Documented implementation

- PostgreSQL operational state, events, idempotency primitives and call/stream correlation.
- Twilio inbound webhook and authenticated bidirectional Media Streams ingress.
- Realtime conversation runtime with provider-neutral interfaces and an OpenAI Realtime adapter.
- Human escalation queue, handoff packet and minimal authenticated Handoff Console scaffold.
- Deterministic capability/authority boundary for external effects.
- Immutable release packaging, staging/preflight and crash-safe cutover-state foundations.

## Evidence boundary

The private project documentation records validation of the foundational, telephony ingress and realtime voice slices, including a controlled live PSTN acceptance test. **This does not establish that the current staged release is deployed to production.**

The documented target release was staged, with production cutover not yet established. A VPS-local deployment adapter/launcher bridge was described as uncommitted and not run on the VPS in the reviewed README.

## Not yet established

- Production-ready Twenty CRM, scheduling or n8n adapters and business side effects.
- Completed production cutover of the documented staged target.
- Production readiness of the Handoff Console.
- Full customer deployment automation.

## Engineering significance

This work demonstrates deliberate separation of AI conversation capability, deterministic authority to perform external actions, durable operational state and controlled release engineering.

## Disclosure

This is a bounded engineering snapshot, **not a production-service certification**. No phone numbers, customer information, credentials, or internal deployment identifiers are published here. Update this case only after new evidence is independently reviewed.
