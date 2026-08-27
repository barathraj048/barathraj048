# Barath M

Founding engineer at [olum.ai](https://olum.ai), where I own the frontend and the backend APIs.
Moving toward backend and distributed systems - currently writing Go and reading DDIA.

[LinkedIn](https://linkedin.com/in/bharath-raj-7992a7248/) ·
[LeetCode](https://leetcode.com/u/barathraj048/) ·
barathraj048@gmail.com

---

### Now

**Founding engineer, olum.ai.** I own the product surface end to end - the frontend and the
backend APIs behind it. Small team, so the API contract, the schema, and what the user sees are
all mine to get right.

- `Olum is an AI-SEO platform that audits a site, then continuously tracks and improves how it surfaces in both Google and AI assistants like ChatGPT - generating the content and fixes to close the gaps.`
- `~70 endpoints across 12 service groups (auth/billing, analysis pipeline, competitor and social intelligence, AI-visibility tracking) behind a 60-page React app, ~79k LOC; long-running crawl/LLM work runs async with client polling, so interactive reads stay under a ~500ms p95 while analysis runs finish in minutes.`
- `Argued for DPoP (RFC 9449) sender-constrained tokens on the payment path instead of plain bearer auth: a non-extractable P-256 keypair in IndexedDB signs a fresh proof per request, and the backend matches its thumbprint against cnf.jkt on the token. Our access token already rides in an httpOnly cookie, so XSS can't read it - but it can still call the API from the victim's page; binding the token to a key JS can never export makes a stolen token useless elsewhere. Cost was a refresh-flow rewrite and a WebCrypto dependency, so I scoped it to payments rather than every route.`

Only put a number here if you measured it. One real number beats three vague ones.

### What I'm learning, in order

Go · DDIA (2nd edition) · storage engines · consensus.

I build APIs at work, so the depth is coming from projects: a key-value store in Go with an
append-only log, CRC-checked records, an index rebuilt from disk on startup, and crash recovery
that survives `kill -9` mid-write. Benchmarks will include the numbers that aren't flattering.

---

### Problem solving

**1,797 contest rating - top 8.4%, ranked 72,294 of 879,441.** 52 contests attended.

329 problems solved: 207 medium, 26 hard. 156 active days in the past year.

---

### Open source

**n8n** *(230k+ active global users)* - five pull requests, none merged. Four were triaged as valid and assigned to internal teams ([#28561](https://github.com/n8n-io/n8n/pull/28561), [#29203](https://github.com/n8n-io/n8n/pull/29203), [#29415](https://github.com/n8n-io/n8n/pull/29415), [#30151](https://github.com/n8n-io/n8n/pull/30151)). What I found: a `Date` serialisation bug in core `deepCopy`, a gap in hybrid dot/bracket notation in the secrets parser, cross-platform path handling that broke Windows CI, and workflow layout coordinates being dropped on save.

Finding real bugs in a 200k-star codebase was the easy half. Landing a patch in a subsystem with an internal owner is the hard half, and I got that wrong five times before I understood why - untested diffs, scope too wide, and writing code before a maintainer had replied. I now comment on the issue and wait for agreement before opening anything.

**Cal.com** *(1M+ signed-in users)* - one merged pull request ([#27251](https://github.com/calcom/cal.com/pull/27251)): a padding fix on `VerticalTabItem`.

### Projects

**In-memory trading engine** - Node.js, Redis, WebSockets. Order matching with an in-memory
book. Load-tested at 1,000 req/s across 500 virtual users: p95 21.9 ms, 0 errors.

**Distributed task pipeline** - Redis producer/consumer with per-task idempotency. Pub/Sub and
WebSocket fan-out instead of HTTP polling. Containerised with Docker.

**Care Ops** - healthcare ERP, built in a 48-hour hackathon (top 10). 367 req/s at 100 virtual
users, no timeouts under load.

---

### Tech

**Languages** TypeScript · JavaScript · Python · Go
**Frontend** Next.js · React · Tailwind
**Backend** Node.js · Express · WebSockets · Pub/Sub · REST API design
**Data** PostgreSQL · Redis · MongoDB
**Infra** Docker · Linux · Vercel

---

### Background

B.E. Electronics & Communication Engineering, P. A. College of Engineering and Technology
(Anna University), 2022–2026.

Student Chairperson, IETE - 18-member team.
