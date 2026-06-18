---
name: agentic-development-discipline
description: Conventions for developing with agentic codegen (Claude Code) under a single-reviewer bottleneck — how to classify a task for supervision depth, what is safe to blind-accept, when to run parallel sessions, how to verify without exhaustive coverage, and how to enforce conventions mechanically. Use whenever working with CC on a real codebase, reviewing AI-generated diffs, deciding whether to read the code or trust the test loop, deciding how much to test, running more than one session at once, or setting up enforcement (lint, CI, branch protection). Trigger even when not stated as "review" — phrases like "running CC," "this diff," "should I write tests for," "two sessions / parallel," "blind accept," "vibe coding," "eval harness," "is this safe to merge/ship," or any question about how closely to supervise generated code should activate this skill.
---

# Agentic Development Discipline

How to keep quality when a machine writes the code and you are the bottleneck. This skill governs **supervision, verification, and enforcement** — it is the complement to architecture conventions (which govern *what* to build). Apply per task, before starting it. Re-read when adopting a new agentic workflow or onboarding someone onto an agent-heavy codebase.

## Core philosophy (applies to every task)

1. **You are a single-reviewer bottleneck.** Throughput is gated by your review capacity, not by codegen speed. The optimization target is attention allocation, never lines produced. Generated code you have not verified is negative value — it is queue, not progress.

2. **Classify every task before starting it.** One axis decides three things at once: how closely to supervise, whether the output is safe to blind-accept, and whether the task can run in parallel. Tagging the task into the wrong lane is the root cause of both wasted supervision and silent defects.

3. **Verification shrinks the space; it does not cover it.** You cannot enumerate the input space or hold the dependency graph in your head — both lose to combinatorial explosion. Reduce what must be checked through structure, not through more test cases.

4. **A convention that can be skipped will be skipped.** Enforce mechanically — lint that fails CI, types that won't compile, branch protection. Social enforcement (hoping people comply, policing diffs, getting angry) is not enforcement; it is the entry point to the failure loop.

5. **Calibrate rigor to stakes, not to anxiety.** The verification a task deserves is a function of blast radius and silence — not of how many edge cases you can imagine.

---

## The classification axis (run before every task)

Every task sits on one axis:

**LANE A — bounded · convention-locked · black-box-testable · loud-when-wrong**
**LANE B — design-sensitive · novel · spec-uncertain · silent-when-wrong**

| | Lane A | Lane B |
|---|---|---|
| Supervision | Glance at behavior (tests, logs, DB) | Read the design *and* the diff |
| Blind-accept | Yes, inside a working test loop | No |
| Parallelizable | Yes | No — single session |
| Overthinking budget | ~Zero | Concentrate it here |

**Lane A examples:** a new CRUD endpoint on an existing pattern, validation against a known schema, frontend wiring to a defined contract, mechanical refactor, test generation, a migration with a hard spec.

**Lane B examples:** a new agentic transaction path, cross-lingual routing logic, anything touching money / auth / data mutation where "correct" is still being discovered.

**Parallelization rule — asymmetric depth only.** If you run two sessions, exactly one is Lane B (gets your attention) and one is Lane A (glanceable). Never two Lane B sessions — you become the bottleneck and quality drops on both. The two streams will not alternate politely; they will both block on input at once, and the one you don't service is either stalled (parallelism gone) or running unsupervised (accumulating plausible-but-wrong code).

**Same project? Pipeline, don't parallelize.** Write the next spec or read ahead while CC runs. Near-zero switch cost because you never leave the one mental model. Two unrelated codebases is maximum switch cost — every switch reloads "where was I, what's the convention here, was that last diff right." Avoid.

**Danger signal:** routing a Lane B task into Lane A *because* you're out of review bandwidth. This is the move that silently poisons a codebase. If you catch yourself doing it, you are over-parallelized — drop a session.

---

## Verification: shrink the space

- **Verify invariants, not scenarios.** A scenario is one point in input space; an invariant is a property over all of it. Ask "what must never happen?" (finite — the real safety properties) instead of "what could happen?" (unbounded — anxiety). A handful of invariants beats infinite test cases and covers inputs you never thought to list.

- **Eliminate cases structurally, not by enumeration.** Caller-agnostic logic, gated handlers, fire-and-forget are the *primary* verification strategy, not fallbacks. A pattern that collapses N cases into 1 invariant beats N tests. This converts an unbounded enumeration problem into a bounded *design* problem — and design you can hold in your head.

- **Hold the seams, not the graph.** Reason about a large system through the contracts between modules, never the whole dependency graph. The layered architecture is working-memory management. When reading generated code, read the seam where the invariant lives — not every path through it.

- **Black-box verification (behavior over code) is valid — inside its envelope.** Trusting the test loop instead of reading the diff is correct when conventions are locked, structure is set, the domain is familiar, and the loop actually exercises the thing. It breaks in exactly three places:
  1. **Coverage gaps you can't see** — on code you haven't read, you don't know which corners exist to test.
  2. **Structural drift** — green tests sit happily on rotting structure; behavior passes while the codebase decays. You pay at the next big change.
  3. **Spec-uncertain work** — you cannot black-box test what you cannot yet specify. This is Lane B. Read it.

- **Budget the overthinking: silent-and-expensive vs loud-and-cheap.** Happy-path bugs are loud, immediate, self-correcting — the feature visibly fails, blast radius is one feature. Spend ~zero anxiety there; let it break loud. Concentrate the entire budget on bugs that are *silent* (wrong state, no crash) AND *expensive* (money, auth, data corruption). That is a small fraction of the surface.

- **Stake-calibrate across projects.** Pre-revenue / pre-PMF: ship light, the bug is cheap, hotfix in minutes — heavy test investment here is over-engineering wearing a verification costume. Deployed / high-stakes (government, money-movement, auth): one genuine audit pass on the critical paths, even if everything else stays vibed.

- **Trace dumps over eval harnesses (until you have users).** For agentic flows pre-PMF, persist run traces (inputs, tool calls, outputs) to inspect by hand — do not build a full eval harness before there's a product to evaluate. The harness is real work that pays off *after* PMF, not before.

---

## Enforcement: mechanical, not social

- **Trusted interior, paid for once at the edge.** Boundary validators, one canonical log format, one response envelope, one outermost catch — push all variance to a few choke points so the interior is uniform and assumption-rich. Downstream code *assumes* validity, so whole classes of case stop existing past the gate. ("Parse, don't validate.")

- **Read the gate, trust the interior.** The more the interior trusts a gate, the higher the stakes of the gate being wrong. A hole in a validator doesn't crash — bad data flows deep and silent. The gate is a silent-expensive risk in the one place you decided to stop looking. Supervision attention goes *to the gate*, not the interior.

- **Make the correct path the only easy path.** `logger.info()` being as easy as `console.log()` is not enforcement — it is a convention with good ergonomics, and it will be skipped. Enforce so the bad path costs active effort: lint that fails CI on `console.log`, a logger that must be imported as the sole sanctioned path, a response type that won't compile if built by hand. Then "they ignored the convention" becomes "their PR went red and they fixed it before review."

- **Legibility, not lock-in.** Gates and envelopes should make the code navigable by *anyone*: learn the boundary, learn the envelope, move through the system without the author present. "Only I can manage this" means the patterns have curdled from leverage into lock-in, and you are now the bus-factor on your own project. Mechanically-enforced patterns are legible without you; taste-enforced patterns need you in the room forever.

- **Gate the repo, not just the code.** Branch protection, no force-push to main, required review, CI that fails on config edits (eslint, tsconfig). Someone disabling a check and force-pushing is not a gap in your enforcement — it is a person routing around a working gate, and the fix is the *merge* gate, not better ergonomics. You cannot tool your way out of someone who won't read the doc, but you can make the bad push impossible to land unreviewed.

---

## Anti-patterns

- **Two Lane B sessions at once.** Doubles producer output against a fixed reviewer; grows the unreviewed queue and drops quality on both.
- **Blind-accepting Lane B work.** Spec-uncertain code trusted to a test loop that cannot specify "correct." This is where silent-expensive bugs are born.
- **Coverage as a completeness strategy.** Enumerating inputs to "fully test" — loses to combinatorial explosion every time. Shrink the space with invariants instead.
- **Social enforcement of conventions.** Policing diffs, hoping for discipline, getting angry. A skippable rule is a skipped rule. Mechanize it.
- **Eval-harness theatre pre-PMF.** Building full evaluation infrastructure for a product with no users. Persist traces; build the harness when there is something to evaluate.
- **"Looks readable" as a quality signal in an unfamiliar language.** You can't distinguish idiomatic from subtly-wrong from working-but-cursed. Behavior passing ≠ code audited; on high-stakes paths, get a real read or a second opinion.

---

## How to use this skill

1. **Classify the task** on the A/B axis *before* starting. Wrong lane is the root error.
2. **Set supervision depth** from the lane — Lane A: glance at behavior; Lane B: read the design and the diff.
3. **Choose verification by stake**, not anxiety — invariants for the silent-expensive paths; loud-and-cheap left to break loud.
4. **Confirm enforcement is mechanical** before relying on it — lint / CI / types / branch protection, not vigilance.
5. **If parallelizing,** verify asymmetric depth (one A, one B max). If you can't, run one session and pipeline the dead time.

The point is not to supervise everything or trust everything. It is to route each task to the right depth, so a single reviewer keeps up without either wasting attention or shipping silent defects.
