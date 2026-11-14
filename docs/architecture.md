# Workflow and engineering decisions

The application connects campaign setup, SMS conversations, booking events, and operator intervention. This document explains the broad engineering choices without disclosing client implementation details or business scripts.

## Keep conversation control explicit

The conversation engine takes the current lead record, an inbound message, and recent history. Intent detection categorizes common responses; state tracks whether the lead is cold, engaged, interested, booked, or unsubscribed.

That separation makes it possible to apply a known response to a familiar situation and use contextual generation selectively. The system includes template fallbacks for AI paths, a short-response limit, and checks intended to reduce repeated automatic replies.

Opt-out tracking and automatic-reply suppression are implementation components. Their presence alone does not establish consent management or compliance across every sending path.

## Persist campaign work

Campaign jobs and pending messages are stored in PostgreSQL rather than held only in application memory. A worker reads pending work in batches, submits messages to the provider, and records outcomes.

This creates a record operators can inspect and supports campaign controls. A production queue also needs reliable scheduling, atomic work claiming, retry decisions, and reconciliation between provider acceptance and stored state. Those are important boundaries when evaluating a particular snapshot.

## Connect a booking to its source conversation

A per-lead token can be attached to a calendar link. The booking handler attempts to resolve the lead using that token, then available email or phone identifiers. Matched bookings update lead state and trigger an operator alert; unmatched bookings can still generate a notification for reconciliation.

This addresses a practical gap between sending a calendar link and understanding which outreach conversation resulted in a booking.

## Give staff a working interface

The dashboard brings lead records, message threads, campaign controls, templates, and summary metrics into one workspace. Manual replies let a staff member continue a conversation directly. Booked leads are excluded from automatic conversation replies.

Template overrides are stored separately from application defaults, allowing routine copy changes through the interface. Optional variants use stable lead assignment so the same lead receives a consistent variant when that selection path is used.

## Separate service responsibilities

| Component | Responsibility |
| --- | --- |
| Dashboard | Campaign setup, lead inspection, copy editing, manual replies, and progress views |
| Express application | Routes, authentication middleware, validation, and service coordination |
| Conversation engine | Intent routing, state decisions, and template/AI response selection |
| Messaging service | Outbound provider requests and inbound webhook integration |
| Database repositories | Lead records, message history, campaign jobs, queue items, and overrides |
| Calendar integration | Booking identifiers, event handling, and lead matching |
| Notifications | Alerts about interest and bookings |

## Cost awareness and observability

The implementation constrains message length and avoids a model call on every reply. These are cost-aware design decisions, not measured savings. Accurate SMS billing also depends on character encoding and provider behavior.

The dashboard exposes sending and engagement summaries. Campaign evaluation needs consistent denominators, complete data retrieval, and reliable attribution before those views can support rigorous conversion or variant-performance claims.

## Deployment considerations

The project includes a local Node.js entry point and serverless deployment configuration. Queue cadence, webhook verification, retry behavior, and dashboard events need to be assessed in the actual deployment environment. The in-process event channel is local to an application instance; polling provides a separate refresh path.
