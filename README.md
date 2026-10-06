# Barath M

Software engineer at [olum.ai](https://olum.ai). I own the product end to end: the React app,
the API layer behind it, and now a Go service for olum.video. The work I care about is the work
that matters under load and failure: idempotency, auth on the money path, and what is still on
disk after a crash.

**Node.js core contributor** · Go · TypeScript · PostgreSQL · Redis · storage engines

[LinkedIn](https://linkedin.com/in/bharath-raj-7992a7248/) ·
[LeetCode](https://leetcode.com/u/barathraj048/) ·
barathraj048@gmail.com

---

### Node.js core

**[nodejs/node#65952](https://github.com/nodejs/node/pull/65952)**: `http: don't destroy socket after request completes`.
Merged. Reviewed by [@mcollina](https://github.com/mcollina) and [@jasnell](https://github.com/jasnell).

**The bug.** If you abort an HTTP request after its response has fully arrived, a process using a
keep-alive agent crashes. One way to trigger it is to abort through `AbortSignal` from inside a
`for await` loop over the response body. Here is the sequence:

1. `ClientRequest.destroy()` called `socket.destroy(err)`, but the `'error'` event fires on a later tick.
2. In that window, `responseKeepAlive()` had already removed the socket's error listener and
   returned the socket to the agent's free pool.
3. The error then landed on a pooled socket with no listener. That is an unhandled `'error'`, so the
   process exited, even though both the request and the response had error handlers.

**The fix.** `destroy()` now skips the socket destroy once the request has finished sending
(`writableFinished`) and the response is complete (`res.complete`), because there is nothing left
to abort. `keepAlive: false` requests already behaved this way, so the fix adds no new behaviour.
It brings the keep-alive path in line with the non-keep-alive one.

---

### Now: olum.ai

Olum is an AI-SEO platform. It audits a site, then keeps tracking and improving how the site shows
up in Google and in AI assistants like ChatGPT, and it generates the content and fixes that close
the gaps. The team is small, so the API contract, the schema and what the user sees are all mine
to get right.

- **API surface.** About 70 endpoints in 12 service groups (auth/billing, analysis pipeline,
  competitor and social intelligence, AI-visibility tracking). They sit behind a 60-page React app
  of about 79k LOC.
- **Fast reads while heavy work runs.** Crawl and LLM jobs run asynchronously and the client polls
  for results. Interactive reads stay under about 500 ms p95, while analysis runs take minutes.
- **Sender-constrained tokens on the payment path.** I argued for DPoP (RFC 9449) instead of plain
  bearer auth. Our access token is already in an httpOnly cookie, so XSS can't read it, but XSS
  can still call the API from the victim's page.
  - **How it works.** A non-extractable P-256 key pair is kept in IndexedDB and signs a fresh proof
    for every request. The backend checks the proof's thumbprint against `cnf.jkt` on the token.
    A stolen token is useless anywhere else because JavaScript can never export the key.
  - **The cost.** It needed a refresh-flow rewrite and a WebCrypto dependency, so I scoped it to
    payments rather than every route.
- **olum.video backend, in Go (in progress).** The service combines a state machine, an
  append-only quota ledger (reserve → commit / void) and caller-supplied idempotency keys. It runs
  on Postgres through `pgx`, with DPoP auth and Redis replay protection. Integration tests run
  against a real Postgres, because the bugs that matter don't exist in a mock: row locks, CHECK
  constraints, and two requests racing for the last video of the day.
  - **What the tests caught.** The idempotency key was globally `UNIQUE`. Two tenants who picked
    the same key would have had the second one handed the first one's reservation, a cross-tenant
    leak. The tests caught it before launch. The constraint is now `UNIQUE (client_id,
    idempotency_key)`, with a regression test.
  - **Done so far.** The schema, auth and quota ledger are done and tested. HTTP handlers are next.

---

### Building: kvgo, a storage engine in Go

kvgo is an embeddable key-value store: a Bitcask engine first, then an LSM tree behind the same
interface. I wrote the design doc before any code. Every decision in it gives a reason and the
alternatives I rejected.

- **Record format.** Writes go to append-only segments. Each record carries a CRC-32C over its
  header, lengths and payload, so a torn or bit-flipped record is detected and never returned as
  data.
- **Explicit durability contract.**
  - `SyncAlways` (the default) fsyncs before acknowledging a write.
  - `SyncBatch` group-commits every 10 ms. It survives `kill -9` but can lose the last interval on
    a power cut.
  - The trade-off is documented up front, not discovered in production.
- **Recovery.** On open, the index is rebuilt from disk and the highest sequence number wins. The
  torn tail of the active segment is cut back to the last good record. Any other damage is
  reported with the file name and offset, never silently skipped.
- **Crash test.** A harness runs `kill -9` mid-write in a loop. After every restart it checks that
  the surviving writes are a prefix of the acknowledged ones.
- **Compaction.** The new segments are written as `.tmp`, fsynced, renamed, and then the directory
  is fsynced. A crash at any step leaves either the old segments or the new ones, never a mix.
- **Benchmarks.** Results are published with the CPU, disk, OS, SyncMode and value sizes next to
  them, including the numbers that aren't flattering.

**Roadmap:** Bitcask (v0.1) → LSM with ordered scans → Raft → buffer pool. I'm reading DDIA
(2nd edition) alongside.

---

### How I work

- **Measured claims only.** A number goes in only if I measured it, and the setup goes next to it.
- **Failure modes before features.** For each change I ask what happens on retry, on a duplicate
  request, on a crash halfway through, and when two callers race.
- **Tests against the real dependency** wherever a mock would hide the bug that matters.
- **Agreement before code** in someone else's codebase. I comment on the issue and wait for a
  maintainer before opening a PR.

---

### Open source

- **Node.js.** One merged PR in `lib/_http_client.js`; details [above](#nodejs-core).
- **n8n** *(230k+ active users).* Five PRs, none merged. Four were triaged as valid and assigned
  to internal teams ([#28561](https://github.com/n8n-io/n8n/pull/28561),
  [#29203](https://github.com/n8n-io/n8n/pull/29203),
  [#29415](https://github.com/n8n-io/n8n/pull/29415),
  [#30151](https://github.com/n8n-io/n8n/pull/30151)). They covered four bugs:
  - `Date` serialisation in core `deepCopy`
  - mixed dot/bracket notation in the secrets parser
  - cross-platform path handling that broke Windows CI
  - workflow layout coordinates dropped on save

  Finding the bugs was the easy half. Landing a patch in a subsystem with an internal owner was the
  hard half. My diffs were untested, too wide in scope, and opened before a maintainer agreed. That
  lesson is why the Node.js PR landed.
- **Cal.com** *(1M+ users).* One merged PR ([#27251](https://github.com/calcom/cal.com/pull/27251)),
  a padding fix on `VerticalTabItem`.

---

### Earlier projects

| Project | Stack | Result |
|---|---|---|
| **In-memory trading engine**: order matching on an in-memory book | Node.js, Redis, WebSockets | Load test at 1,000 req/s with 500 virtual users: p95 21.9 ms, 0 errors |
| **Distributed task pipeline**: Redis producer/consumer with per-task idempotency; Pub/Sub and WebSocket fan-out instead of HTTP polling | Node.js, Redis, Docker | Containerised |
| **Care Ops**: healthcare ERP built in a 48-hour hackathon (top 10) | Node.js | 367 req/s at 100 virtual users, no timeouts |

---

### Problem solving

**LeetCode contest rating 1,797, top 8.4%** (ranked 72,294 of 879,441) across 52 contests.
329 problems solved (207 medium, 26 hard), with 156 active days in the past year.

---

### Tech

**Languages:** Go · TypeScript · JavaScript · Python<br>
**Backend:** Node.js · Go `net/http` · Express · WebSockets · Pub/Sub · REST API design<br>
**Data:** PostgreSQL (`pgx`) · Redis · MongoDB<br>
**Security:** DPoP (RFC 9449) · JWT · WebCrypto<br>
**Frontend:** Next.js · React · Tailwind<br>
**Infra:** Docker · Linux · Vercel

---

### Background

B.E. Electronics & Communication Engineering, P. A. College of Engineering and Technology
(Anna University), 2022–2026. Student Chairperson of IETE, leading an 18-member team.
