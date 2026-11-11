# ReviewOps walkthrough

## What to demonstrate

Human-in-the-loop is a real persisted state machine. The model or UI cannot grant itself reviewer authority. Approval creates a concrete immutable export; rejection creates no export.

### Walk through it

1. Log in as Alice and enter the exact report proposed for export.
2. Submit with a unique idempotency key. Observe pending status/version.
3. Restart the API while it is pending.
4. Switch to the development reviewer credential. Inspect the exact proposal, then approve.
5. Return to Alice and download the actual artifact.
6. Try rejection on another proposal; there must be no artifact.
7. Try approval as an analyst, a stale version, and reuse of the same key with altered content. These must fail.
8. Repeat the identical approved decision. The stored outcome is returned without a second export.

### Why it works

The [review graph](../src/portfolio/review.py) uses native interrupt/resume with a SQLite checkpointer. The [storage transaction](../src/portfolio/storage.py) verifies tenant/role/version, commits the business decision and artifact together, and records an audit row. Resume rereads that committed decision. Side effects are not placed before the interrupt.

This example exports a local report blob. It does not send email, charge a payment method or publish online. To add any such effect, use an outbox/worker, a narrowly scoped integration credential, a stable operation idempotency key and observable delivery status.

### Compare LangChain and LangGraph HITL

Run the Foundation Studio middleware demo too. Native graph interrupts pause arbitrary graph work; HumanInTheLoopMiddleware pauses configured tool calls inside create_agent. Both need matching checkpoint/thread continuity. ReviewOps adds business authorization and transactional effects beyond the pause mechanism.

### Practice extensions

Implement an edit decision by creating a new immutable proposal version, showing the final edited payload for review, and binding approval to its fingerprint. Add a second information-request interrupt with stable ordering. Add a test that a forged resume payload cannot bypass the business decision.
