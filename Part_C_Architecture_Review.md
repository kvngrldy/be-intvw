# Part C: Architecture Review (Candidate)

Audience: Candidate
Description: Architecture diagram, flow, and constraints. Export as PDF for candidates.
Section: Part C

You are looking at the architecture for a payment disbursement system at a fintech company. Review the system design and walk us through what concerns you.

Use whatever tools you normally use. Think out loud as you reason through the architecture.

---

## System Diagram

```
┌─────────┐     ┌──────────────────┐     ┌──────────────────────────────┐
│  User   │────▶│  Express Server  │────▶│  Backend Service (Rails)     │
│(Browser)│     │  (JWT + routing) │     │  - Transaction management    │
└─────────┘     └──────────────────┘     │  - Compliance checks         │
                                         │  - Disbursement orchestration│
                                         └──────────┬───────────────────┘
                                                    │
                         ┌──────────────────────────┼──────────────────────┐
                         │                          │                      │
                         ▼                          ▼                      ▼
              ┌─────────────────┐       ┌────────────────┐      ┌──────────────────┐
              │   Primary DB    │       │     Redis      │      │  Payment Provider│
              │   (AWS RDS)     │       │  - Cache       │      │  (External API)  │
              │                 │       │  - Idempotency │      │                  │
              │  Stores:        │       │  - Job queues  │      │  Executes actual │
              │  - Users        │       │                │      │  bank transfers  │
              │  - Payments     │       │  Expiry: 10min │      │                  │
              │  - Transactions │       │  for idemp.    │      │  Rate limited    │
              └────────┬────────┘       └────────────────┘      │  Charges per call│
                       │                                        └──────────────────┘
                       ▼
              ┌─────────────────┐
              │  Read Replicas  │
              │  (AWS RDS)      │
              │                 │
              │  Lag: up to 2s  │
              │  Used for:      │
              │  - Dashboard    │
              │  - Reports      │
              │  - API reads    │
              └─────────────────┘
```

---

## Flow: User Initiates a Withdrawal

1. User submits withdrawal request via browser
2. Express server validates JWT, forwards to backend
- Backend runs compliance checks (KYC, sanctions screening)
1. On compliance pass, backend calls Payment Provider API to execute disbursement
2. Payment Provider returns status ("processing" or error)
3. Backend updates payment record in Primary DB
4. Redis stores idempotency key (10-minute TTL) to prevent duplicate processing
5. Read replicas serve subsequent status queries from the user dashboard

---

## Constraints

- Payment Provider has a 99.5% SLA and occasional 5-30 second timeouts
- Peak load: ~500 requests/second
- Compliance checks take 200ms-2s depending on sanctions list provider
- Regulatory requirement: all transactions must be auditable with full history
- Redis is a single node (no cluster) with persistence disabled

---

## Your Task

You have 35 minutes. Walk us through:

1. What concerns you about this architecture?
2. Where are the highest-risk failure points?
3. What would you change or add?