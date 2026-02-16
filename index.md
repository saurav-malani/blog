# The 5-Decision Framework: How to Master Low-Level System Design

You've read the source code. You've studied the architecture docs. You've watched the system design videos. And yet, when you sit down to design something from scratch — the elegance doesn't come.

The problem isn't knowledge. It's that you're looking at finished systems and trying to reverse-engineer genius. That's like watching a chess grandmaster's final move and asking "how did they see that?" You're starting from the wrong end.

Elegant low-level design doesn't come from imagining the whole system. It comes from making 5 decisions well. This post is about those 5 decisions, and a deliberate practice framework to internalize them.

---

## The Insight That Changed How I Think About LLD

Consider two systems that seem completely different:

- **GDB** — debugs native programs (C, C++, Rust)
- **Chrome DevTools Protocol (CDP)** — debugs web pages (HTML, CSS, JS)

Different domains. Different eras. Different teams. But strip away the surface, and the design is identical:

```
A running system (opaque)
    │
    exposes a protocol
    │
    ▼
An external tool can now:
    1. Observe state    (inspect variables / inspect DOM)
    2. Pause execution  (breakpoints)
    3. Modify state     (change a variable / modify DOM)
    4. Control flow     (step through / navigate)
```

Both are an **introspection protocol** — a standardized way for an external observer to see and control the internals of a running system. The same pattern appears in OpenTelemetry (distributed systems), database wire protocols (Postgres), and container runtimes (Docker).

The designers of these systems didn't start with "let me build an introspection protocol." They started with a pain: "I can't see what's happening inside this thing." And then they made 5 decisions.

---

## The 5 Decisions

Every low-level design, no matter how complex, boils down to these decisions made in order:

### D1: Boundaries — What are the separate things?

Every system is a conversation between two or more distinct parts. Before anything else, draw the line.

CDP: DevTools (the inspector) and Chrome engine (the thing being inspected). Two sides.

Git: Working directory, staging area, local repo, remote repo. Four boundaries.

Redis: Client and a single-threaded server. Two sides.

The question is: **where do you draw the walls?** Too few boundaries and you get a monolith. Too many and you get unnecessary complexity. The right boundaries usually follow the lines of **different rates of change** — things that change together belong together, things that change independently should be separated.

### D2: Contracts — How do they talk to each other?

Once you have boundaries, the parts need to communicate. This is the interface between them.

CDP chose: JSON messages over WebSocket.
GDB chose: text commands over a serial/socket connection.
HTTP chose: text request → text response over TCP.
Postgres chose: binary messages over TCP.

The choice of contract determines almost everything about what's easy and hard to build on top. JSON over WebSocket means any language can use CDP trivially. GDB's text protocol means you can literally debug by typing commands manually into a terminal.

**A good contract is one that you can test with the simplest possible tool.** If you can test your protocol with `curl` or `telnet` or `echo`, it's probably well-designed.

### D3: Primitives — What's the smallest unit?

This is the most important decision. The primitive is the atom of your system — the smallest indivisible unit that everything else is built from.

| System | Primitive | Why it works |
|--------|-----------|-------------|
| Git | The commit (snapshot + parent pointer + message) | Immutable, content-addressed, forms a DAG naturally |
| CDP | The method call `{ method, params } → { result }` | Simple request-response, extensible by adding methods |
| Redis | The key-value pair | Everything reduces to "store this, retrieve this" |
| Docker | The layer (one filesystem diff per instruction) | Composable, cacheable, shareable |
| HTTP | The request (verb + path + headers + body) | Stateless, cacheable, universally understood |

The primitive is what makes a design elegant or clunky. If you choose the right primitive, the rest of the system almost designs itself. If you choose the wrong one, you'll fight it forever.

**How to evaluate a primitive:** ask "does this compose?" If you can combine two of them to make something useful, it's probably right. Git commits compose into branches. Docker layers compose into images. HTTP requests compose into APIs. Good primitives are like LEGO blocks.

### D4: Flow — What triggers what? In what order?

Now you have boundaries, contracts, and primitives. Flow is the choreography — what happens in what sequence.

CDP: Inspector sends command → Chrome executes → Chrome returns result. Chrome can also push events (like "page loaded") at any time.

Git: Edit → stage → commit → push. A linear pipeline with clear stages.

Redis: Client sends command → Redis executes (single-threaded, no concurrency) → returns result.

Flows tend to be one of three patterns:

```
Request-Response:  A asks B, B answers     (HTTP, Redis, CDP commands)
Pipeline:          data flows A → B → C    (Git staging, Unix pipes)
Event-driven:      B notifies A when       (WebSocket events, CDP events,
                   something happens         pub/sub systems)
```

Most real systems combine two or three of these. CDP is primarily request-response but also event-driven. Understanding which pattern dominates tells you a lot about the system's character.

### D5: Failure — What breaks? What happens when it does?

This is the decision most people skip in practice and regret in production.

Every system has a small number of things that can go wrong. The design must account for each:

| System | What breaks | How it handles it |
|--------|------------|-------------------|
| CDP | Tab crashes | Error event, inspector can reconnect |
| Git | Merge conflict | Stops and asks human to resolve |
| Redis | Out of memory | Eviction policies (LRU, LFU, etc.) |
| HTTP | Network timeout | Status codes, retry semantics |
| Postgres | Transaction deadlock | Detects and aborts one transaction |

**The elegance test for failure handling:** does the system fail in a way that's consistent with its primitives? Git doesn't silently merge conflicts — it stops and presents the conflict as a diff (its primitive). Redis doesn't crash on OOM — it evicts keys (its primitive). When failure handling feels alien to the core design, something is wrong.

---

## The Practice Framework

Understanding the 5 decisions is the map. Deliberate practice is the territory. Here's a structured program to build real intuition.

### Phase 1: Decompose Existing Systems (Weeks 1–4)

Don't design anything yet. Take systems you already use and reverse-engineer the 5 decisions.

**The exercise:** Pick a system. Write down D1–D5. Then go read the actual source or docs. Compare what you assumed with what was actually built.

**Week 1 — Protocols:**

```
HTTP
  D1: Client and Server
  D2: Text request → text response over TCP
  D3: The request (verb + path + headers + body)
  D4: Client sends → Server processes → Server responds
  D5: Timeout, malformed request → status codes (404, 500)

WebSocket
  D1: Two peers (originally client + server, then symmetric)
  D2: Bidirectional frames over persistent TCP connection
  D3: The frame (a small message, text or binary)
  D4: Either side sends anytime (event-driven)
  D5: Connection drop → close frame, heartbeat ping/pong

After both: Why does WebSocket exist when HTTP exists?
Because HTTP's primitive (request-response) couldn't model
"server pushes to client." The PRIMITIVE was wrong for the
use case. That forced a new protocol.
```

**Week 2 — Tools:**

```
Git
  D1: Working dir, staging area, local repo, remote repo
  D2: Commands that move data between boundaries
  D3: The commit (snapshot + parent + message)
  D4: Edit → stage → commit → push
  D5: Merge conflicts, network failure, corrupt objects

Docker
  D1: Image (blueprint) and Container (running instance)
  D2: Dockerfile defines image, docker run creates container
  D3: The layer (each instruction = one filesystem diff)
  D4: Build layers → create image → run container → expose ports
  D5: Build failure, port conflicts, OOM kills
```

**Week 3 — Databases:**

```
Redis
  D1: Client and single-threaded server
  D2: RESP text protocol over TCP
  D3: The key-value pair
  D4: Send command → execute → return result
  D5: OOM (eviction), persistence failure (RDB/AOF)

PostgreSQL
  D1: Client, parser, planner, executor, storage engine
  D2: Wire protocol (binary messages over TCP), SQL as contract
  D3: The tuple (row) and relation (table)
  D4: Parse → Plan → Execute → Return rows
  D5: Deadlock, transaction abort, disk full
```

**Week 4 — Introspection systems:**

```
CDP (Chrome DevTools Protocol)
  D1: DevTools UI and Chrome engine
  D2: JSON messages over WebSocket
  D3: The method call { method, params } → { result }
  D4: Inspector sends → Chrome executes → returns result;
      Chrome pushes events
  D5: Tab crash, method not found, timeout

OpenTelemetry
  D1: Application (instrumented) and Collector (aggregator)
  D2: gRPC/HTTP protocol
  D3: The span (one unit of work with timing + context)
  D4: App creates spans → exports to collector → collector
      routes to backend
  D5: Collector down (buffer/drop), high cardinality explosion
```

**After each exercise, write down:**
1. Which decision was the HARDEST for the designers? Why?
2. Which primitive (D3) is the most elegant? Why does it survive?
3. Where do I see the SAME pattern as a previous system?

That third question is how pattern recognition builds.

### Phase 2: Redesign From Scratch (Weeks 5–8)

Pretend the system doesn't exist. Start from only the pain. Design it yourself. Then compare.

**The exercise:** Write D1–D5 for a system you've never built, starting only from the problem it solves. Do NOT look at the real solution until you're done.

```
Week 5: "I need to debug a program running on another machine."

  Start from the pain. What are the two sides? How do they
  talk? What's the smallest operation? What's the flow?
  What breaks?

  Design it. Then read how GDB actually works. Compare.
  Where did you match? Where did you differ? Why?

Week 6: "I need web apps to store data locally in the browser."

  Design localStorage / IndexedDB from scratch.
  Then compare with the real APIs.

Week 7: "I need to isolate processes on one machine."

  Design containers from scratch.
  Then compare with cgroups + namespaces + overlay filesystem.

Week 8: "I need services to tell me what they're doing in production."

  Design a distributed tracing system from scratch.
  Then compare with OpenTelemetry.
```

The gap between your design and reality is where the learning lives. Don't judge yourself for the gap — study it.

### Phase 3: The Primitive Test (Weeks 9–12)

This phase builds the deepest intuition. For each system you've studied, change the primitive and trace the consequences.

**The exercise:** Take a system. Change one decision (usually D3, the primitive). Follow the ripple effects.

```
CDP uses JSON messages over WebSocket.
  What if it used binary protobuf instead?
    → harder to debug manually, faster for high-frequency tools
    → the ecosystem of simple tools wouldn't have grown as fast
  What if it was file-based instead of socket-based?
    → no real-time debugging, only post-mortem analysis
    → completely different use cases emerge

Git's primitive is a commit (snapshot + parent pointer).
  What if it stored diffs instead of snapshots?
    → reading any file requires replaying all diffs from the start
    → branching becomes much more expensive
    → this is actually what older VCS (SVN, Perforce) did
  What if commits didn't have parent pointers?
    → no history, no branches, no merge — just detached snapshots
    → it becomes a backup tool, not a version control system

Redis's primitive is a key-value pair.
  What if keys could be hierarchical (like a filesystem)?
    → you get etcd/ZooKeeper — a different tool for a different problem
  What if values could only be strings?
    → you lose lists, sets, sorted sets — Redis becomes memcached
```

Each change cascades through the entire system. Tracing these cascades teaches you how **one primitive decision shapes everything above it**. After a month of this, you'll start feeling which decisions are "load-bearing" and which are incidental.

### Phase 4: Design Original Systems (Ongoing)

Now you're ready to design from zero. Pick real problems.

**Easy (single-boundary systems):**
- Design a rate limiter
- Design a URL shortener
- Design a job queue
- Design a cache with TTL

**Medium (multi-boundary systems):**
- Design a real-time collaborative editor
- Design a feature flag system
- Design a notification system (email + push + in-app)
- Design a plugin/extension system for an application

**Hard (systems where the primitive choice is non-obvious):**
- Design a distributed cache with invalidation
- Design an AI agent tool protocol
- Design a browser extension ↔ native app communication bridge
- Design a conflict-free replicated data type (CRDT)

For each, use D1–D5 strictly. Write it down. Research how real systems solved the same problem. Compare and journal.

---

## The Journal

Keep a running log. This is the most valuable artifact you'll produce.

```markdown
### [date] — System: Redis

D1: Client + single-threaded server
D2: RESP text protocol over TCP
D3: Key-value pair
D4: Command → Execute → Respond
D5: OOM → eviction policies

Insight: Single-threaded was the key design choice.
Eliminates ALL concurrency bugs. The constraint IS the
elegance. Sometimes the best design decision is what
you choose NOT to support.

Pattern match: Same as Node.js event loop — choose
simplicity over parallelism, scale horizontally instead.
```

Over months, your journal becomes a personal pattern library. The **insights** and **pattern matches** across systems are where expertise lives. Knowledge is knowing how Redis works. Expertise is recognizing that Redis, Node.js, and Nginx all made the same bet — and understanding why.

---

## Accelerators

### Read Small, Elegant Codebases

Don't read Chrome (millions of lines). Read systems small enough to hold in your head:

- **Redis** (~30K lines of C) — arguably the most elegant server ever built
- **SQLite** (~150K lines but one file, one of the most deployed software in history)
- **Lua** (~30K lines) — tiny language, beautifully designed VM
- **BoltDB** (~5K lines of Go) — key-value store you can read in a weekend

These are small enough to read in days but real enough to teach you how elegant primitives look in actual code.

### The "Two Things" Shortcut

When you're stuck on any LLD problem, ask:

> "What are the two things that need to talk, and what's the simplest contract between them?"

This single question cuts through 90% of the complexity. There are almost always exactly two sides. Find them, define how they talk, and the rest follows.

### The Composition Test

When evaluating your design, ask:

> "Can I combine two of my primitives to make something useful?"

If yes, your primitive is probably right. Git commits compose into branches. Docker layers compose into images. HTTP requests compose into REST APIs. Unix commands compose via pipes. Good primitives are LEGO blocks — useful alone, powerful together.

### The Absence Test

When evaluating what to include, ask:

> "If I remove this, does the system still make sense?"

Keep removing until it breaks. What's left after that process is the essential design. Everything else is optimization, convenience, or future-proofing — add it later, not now.

---

## The Takeaway

Low-level design mastery isn't about memorizing how systems work. It's pattern recognition — seeing that CDP, GDB, and OpenTelemetry are the same design in different costumes. That Redis, Node.js, and Nginx made the same fundamental bet. That Git and Docker both chose immutable, composable primitives and reaped the same benefits.

The 5 decisions — Boundaries, Contracts, Primitives, Flow, Failure — are the lens. The practice framework — decompose, redesign, test primitives, design original, journal — is how you grind the lens until it's sharp.

Start this week. Pick one system you use daily. Write down D1–D5. You'll be surprised how much you already understand — and how much sharper that understanding becomes when you make it explicit.
