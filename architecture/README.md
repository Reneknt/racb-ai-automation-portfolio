# Architecture Evidence

The architectural explanations and diagrams are maintained alongside each project case study so that design choices can be read together with their implementation and validation boundaries.

## Architecture by system

| System | Architecture focus |
| --- | --- |
| [Agent Factory](../projects/racb-agent-factory.md#system-architecture) | Governed multi-agent delivery, review separation, deterministic Git gates and durable state |
| [n8n Workflow Factory](../projects/racb-n8n-workflow-factory.md) | Workflow artifact validation, compatibility and controlled automation |
| [OperatorOS](../projects/racb-operatoros.md) | Human-controlled operations, execution boundaries and evidence |
| [Homelab](../projects/homelab.md) | Versioned infrastructure, container services and observability |
| [Production VPS](../projects/racb-production-vps.md) | Production hosting, hardening and recovery validation |
| [RACBCONSULTING MVP](../projects/racbconsulting-mvp.md) | Assessment application, authorization and proposal workflow |
| [RACB Connect](../projects/racb-connect.md) | Static-first conversion gateway and handoff |
| [Turno](../projects/turno.md) | Multi-tenant scheduling and service boundaries |
| [RACB Voice Platform](../projects/racb-voice-platform.md) | Telephony ingress, realtime voice and deterministic authority |
| [RACB Content Platform](../projects/racb-content-platform.md) | Content workflows and publishing validation boundaries |
| [AgentConfig Auditor](../projects/agentconfig-auditor.md#extensible-architecture) | Deterministic preflight rule engine |

The diagrams are explanatory representations, not exports of private production topology. They do not imply that all depicted components are deployed or production-validated. See each case for its specific maturity and evidence classification.

Sensitive host details, credentials, customer information and proprietary implementation remain outside this public repository.
