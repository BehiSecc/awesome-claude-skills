---
name: fp-check
description: "Systematically verifies suspected security bugs to eliminate false positives. Produces TRUE POSITIVE or FALSE POSITIVE verdicts with documented evidence for each bug."
allowed-tools:
  - Read
  - Grep
  - Glob
  - LSP
  - Bash
  - Task
  - Write
  - Edit
  - AskUserQuestion
  - TaskCreate
  - TaskUpdate
  - TaskList
  - TaskGet
---

# False Positive Check

## When to Use

- "Is this bug real?" or "is this a true positive?"
- "Is this a false positive?" or "verify this finding"
- "Check if this vulnerability is exploitable"
- Any request to verify or validate a specific suspected bug

## When NOT to Use

- Finding or hunting for bugs ("find bugs", "security analysis", "audit code")
- General code review for style, performance, or maintainability
- Feature development, refactoring, or non-security tasks
- When the user explicitly asks for a quick scan without verification

## Rationalizations to Reject

If you catch yourself thinking any of these, STOP.

| Rationalization | Why It's Wrong | Required Action |
|---|---|---|
| "Rapid analysis of remaining bugs" | Every bug gets full verification | Return to task list, verify next bug through all phases |
| "This pattern looks dangerous, so it's a vulnerability" | Pattern recognition is not analysis | Complete data flow tracing before any conclusion |
| "Skipping full verification for efficiency" | No partial analysis allowed | Execute all steps per the chosen verification path |
| "The code looks unsafe, reporting without tracing data flow" | Unsafe-looking code may have upstream validation | Trace the complete path from source to sink |
| "Similar code was vulnerable elsewhere" | Each context has different validation, callers, and protections | Verify this specific instance independently |
| "This is clearly critical" | LLMs are biased toward seeing bugs and overrating severity | Complete devil's advocate review; prove it with evidence |

---

## Step 0: Understand the Claim and Context

Before any analysis, restate the bug in your own words. If you cannot do this clearly, ask the user for clarification using AskUserQuestion. Half of false positives collapse at this step — the claim doesn't make coherent sense when restated precisely.

Document:

- **What is the exact vulnerability claim?** (e.g., "heap buffer overflow in `parse_header()` when `content_length` exceeds 4096")
- **What is the alleged root cause?** (e.g., "missing bounds check before `memcpy` at line 142")
- **What is the supposed trigger?** (e.g., "attacker sends HTTP request with oversized Content-Length header")
- **What is the claimed impact?** (e.g., "remote code execution via controlled heap corruption")
- **What is the threat model?** What privilege level does this code run at? Is it sandboxed? What can the attacker already do before triggering this bug? (e.g., "unauthenticated remote attacker vs privileged local user"; "runs inside Chrome renderer sandbox" vs "runs as root with no sandbox")
- **What is the bug class?** Classify the bug and consult [bug-class-verification.md]({baseDir}/references/bug-class-verification.md) for class-specific verification requirements that supplement the generic phases below.
- **Execution context**: When and how is this code path reached during normal execution?
- **Caller analysis**: What functions call this code and what input constraints do they impose?
- **Architectural context**: Is this part of a larger security system with multiple protection layers?
- **Historical context**: Any recent changes, known issues, or previous security reviews of this code area?

## Route: Standard vs Deep Verification

After Step 0, choose a verification path.

### Standard Verification

Use when ALL of these hold:

- Clear, specific vulnerability claim (not vague or ambiguous)
- Single component — no cross-component interaction in the bug path
- Well-understood bug class (buffer overflow, SQL injection, XSS, integer overflow, etc.)
- No concurrency or async involved in the trigger
- Straightforward data flow from source to sink

Follow [standard-verification.md]({baseDir}/references/standard-verification.md). No task creation — work through the linear checklist, documenting findings inline.

### Deep Verification

Use when ANY of these hold:

- Ambiguous claim that could be interpreted multiple ways
- Cross-component bug path (data flows through 3+ modules or services)
- Race conditions, TOCTOU, or concurrency in the trigger mechanism
- Logic bugs without a clear spec to verify against
- Standard verification was inconclusive or escalated
- User explicitly requests full verification

Follow [deep-verification.md]({baseDir}/references/deep-verification.md). Create the full task dependency graph and execute phases with the plugin's agents.

### Default

Start with standard. Standard verification has two built-in escalation checkpoints that route to deep when complexity exceeds the linear checklist.

## Batch Triage

When verifying multiple bugs at once:

1. Run Step 0 for all bugs first — restating each claim often collapses obvious false positives immediately
2. Route each bug independently (some may be standard, others deep)
3. Process all standard-routed bugs first, then deep-routed bugs
4. After all bugs are verified, check for **exploit chains** — findings that individually failed gate review may combine to form a viable attack

## Final Summary

After processing ALL suspected bugs, provide:

1. **Counts**: X TRUE POSITIVES, Y FALSE POSITIVES
2. **TRUE POSITIVE list**: Each with brief vulnerability description
3. **FALSE POSITIVE list**: Each with brief reason for rejection

## References

- [Standard Verification]({baseDir}/references/standard-verification.md) — Linear single-pass checklist for straightforward bugs
- [Deep Verification]({baseDir}/references/deep-verification.md) — Full task-based orchestration for complex bugs
- [Gate Reviews]({baseDir}/references/gate-reviews.md) — Six mandatory gates and verdict format
- [Bug-Class Verification]({baseDir}/references/bug-class-verification.md) — Class-specific verification requirements for memory corruption, logic bugs, race conditions, integer issues, crypto, injection, info disclosure, DoS, and deserialization
- [False Positive Patterns]({baseDir}/references/false-positive-patterns.md) — 13-item checklist and red flags for common false positive patterns
- [Evidence Templates]({baseDir}/references/evidence-templates.md) — Documentation templates for data flow, mathematical proofs, attacker control, and devil's advocate reviews
