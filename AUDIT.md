# Clodds Codebase Audit - Feb 5 2026

## What We Just Fixed
- **LanceDB local embeddings** - `src/extensions/memory-lancedb/index.ts` now uses `getTransformersPipeline()` from `src/embeddings/index.ts` instead of hash stub. Dimension default fixed (384 for local, 1536 for OpenAI/Cohere).

---

# Full Repository Analysis - Mar 15 2026

## Overview

| Metric | Result |
|--------|--------|
| TypeScript errors | ✅ 0 |
| Test suite | ✅ 135/135 pass |
| npm vulnerabilities (before) | ⚠️ 17 (1 critical, 9 high, 6 moderate, 1 low) |
| npm vulnerabilities (after) | ✅ 5 moderate (unfixable, see below) |
| Source files | 168k+ lines across 95+ modules |
| Bundled skills | 121 |
| Messaging channels | 21 |
| Test files | 31 |

---

## Security Fixes Applied (Mar 15 2026)

The following vulnerabilities were resolved by updating `package.json` overrides and direct dependencies:

| Package | Severity | Vulnerability | Fix |
|---------|----------|---------------|-----|
| `undici` | High | WebSocket 64-bit length overflow + HTTP smuggling (≤6.23.0) | Override `^6.23.0` → `^6.24.1` |
| `minimatch` | High | ReDoS via multiple GLOBSTAR segments (10.0.0–10.2.2) | Override `^10.2.1` → `^10.2.4` |
| `basic-ftp` | Critical | Path traversal in `downloadToDir()` (<5.2.0) | Added override `^5.2.0` |
| `flatted` | High | Unbounded recursion DoS in `parse()` (<3.4.0) | Added override `^3.4.0` |
| `serialize-javascript` | High | RCE via `RegExp.flags` and `Date.prototype.toISOString()` (≤7.0.2) | Added override `^7.0.3` |
| `underscore` | High | Unlimited recursion DoS in `_.flatten`/`_.isEqual` (≤1.13.7) | Added override `^1.13.8` |
| `file-type` | Moderate | Infinite loop in ASF parser + ZIP decompression bomb (13.0.0–21.3.1) | Added override `^21.3.2` |
| `fast-xml-parser` | Low | Stack overflow in XMLBuilder with preserveOrder (5.0.0–5.3.7) | Updated direct dep `^5.3.7` → `^5.5.5` |

### Remaining Vulnerabilities (Unfixable)

5 moderate vulnerabilities remain in `puppeteer`'s transitive dependency chain:

- `yauzl@2.10.0` (off-by-one error, fix requires yauzl ≥ 3.2.1)
- `extract-zip@2.0.1` (via yauzl ≤ 2.x)
- `@puppeteer/browsers@2.13.0` (via extract-zip)
- `puppeteer@24.37.5` / `puppeteer-core@24.37.5`

**Root cause:** `extract-zip@2.x` requires `yauzl@^2.x` while the fix is in the incompatible `yauzl@3.x`. Even the latest `puppeteer@24.39.1` still ships `@puppeteer/browsers@2.13.0`. These cannot be overridden without breaking the ZIP extraction API that `extract-zip` and `@puppeteer/browsers` rely on. **Impact is low** — `puppeteer` is a dev-only dependency used exclusively for PDF export; the `yauzl` vulnerability is only exploitable when processing attacker-controlled ZIP files, which does not occur in Clodds PDF rendering.

---

## New Modules Added (Mar 15 2026)

Four new source modules were added to the codebase. All compile cleanly (0 TypeScript errors) and are fully integrated:

### `src/acp/` — Agent Control Protocol

Multi-agent orchestration system providing:
- `AgentRegistry` — register/unregister agents with heartbeat monitoring
- `TaskQueue` — priority-based task delegation with timeout support
- `MessageBus` — publish/subscribe inter-agent messaging
- `ACP` orchestrator — load balancing strategies (round-robin, least-busy, capability)
- Files: `index.ts`, `registry.ts`, `identity.ts`, `agreement.ts`, `escrow.ts`, `predictions.ts`, `discovery.ts`, `persistence.ts`

### `src/web/` — Embeddable Web Server

Lightweight HTTP/WebSocket server providing:
- HTTP + HTTPS (TLS) server with configurable auth (basic, bearer, none)
- WebSocket server with session management and secure message validation
- Route registration API (GET, POST, PUT, DELETE, PATCH)
- CORS support, static file serving, rate limiting hooks
- File: `index.ts`

### `src/wizard/` — Onboarding Wizard

Step-by-step CLI configuration wizard:
- `createOnboardingWizard()` factory with pluggable steps
- Built-in steps: welcome, Anthropic API key, Telegram, Polymarket, finish/save
- Writes `.env` file with collected credentials
- File: `index.ts`

### `src/workspace/` — Workspace Management

Project context detection and AI prompt injection:
- `WorkspaceManager` — detects workspace root (git, node, python, generic)
- Loads `AGENTS.md`, `SOUL.md`, `CLAUDE.md`, `README.md`, `.clodds.json`
- Builds layered system prompts (user-level + project-level)
- `createAgentsMd()`, `createSoulMd()` scaffolding helpers
- Bounded directory walk (max 10,000 files, depth 3) with path-traversal protection
- File: `index.ts`

---

## P1 - High Priority

(All P1 items complete)

---

## P2 - Medium Priority

### 2. Test Coverage (Verified)
- **31 test files** in `tests/` directory (was 25, now 31)
- **135 tests pass** (100%)
- Covers: command parsing, risk guards, ledger, market-index, webhooks, trading safety, HTTP rate limiting, ML pipeline, backtesting, signal router, webhook security
- Run `npm test` for full suite

---

## P3 - Nice to Have

(All P3 items complete)

---

## Done (Reference)
- [x] LanceDB local embeddings wired to transformers.js pipeline
- [x] Pipeline type and function exported from embeddings service
- [x] Dimension default fixed for local model (384-dim)
- [x] Kamino SDK installed (klend-sdk@2.10.6, kliquidity-sdk@6.0.0 — web3.js v1 compatible)
- [x] Fixed all Kamino API mismatches: APY→APR, stats→state, getTransactions, build*Txns signatures, vault deposit/withdraw/holders
- [x] **0 TypeScript errors** — clean typecheck
- [x] Futures: setLeverage, setMarginType, getIncomeHistory, cancelOrder (all 4 platforms)
- [x] Polymarket/Kalshi retry with exponential backoff
- [x] Atomic nonce generation (BigInt counter)
- [x] Kalshi slippage protection + polling-based triggers
- [x] **PDF export** via puppeteer (open-prose extension)
- [x] **Trade executor** added Drift, Bybit, MEXC platforms
- [x] **DOCX export/import** via docx (write) + mammoth (read)
- [x] **Embeddings skill** properly wired to createDatabase() with async init
- [x] **Channel adapters audited** - 15/20 production ready (Discord, Slack, Telegram, WhatsApp, Teams, Signal, Matrix, LINE, iMessage, Mattermost, Twitch, Nostr, BlueBubbles, Nextcloud Talk, Tlon). Partial: Google Chat, WebChat, Voice, Zalo Personal
- [x] **Deprecated field cleanup** - Removed redundant `replyToMessageId` from WhatsApp (uses thread.replyToMessageId)
- [x] **Task runner audit logging** - Added comprehensive logging to shell and file executors (start, complete, fail events)
- [x] **Security fixes (Mar 2026)** — Fixed 12 vulnerabilities (1 critical, 9 high, 2 moderate, 1 low) via package.json overrides; 5 moderate remain in puppeteer's unfixable transitive chain
- [x] **New modules (Mar 2026)** — `src/acp/`, `src/web/`, `src/wizard/`, `src/workspace/` — all compile cleanly, 0 errors
- [x] **Test suite expanded** — 135 tests across 31 files (100% pass rate)
