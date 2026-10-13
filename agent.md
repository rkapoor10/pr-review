# AGENTS.md

> **Repository Architecture & Autonomous Agent Protocols**  
> **Version:** 2.4.0 | **Last Updated:** 2026-10-12 | **Status:** Active

This document defines the working guidelines, operational boundaries, and system architecture for autonomous AI agents (including Claude, Copilot, and custom LLM runners) operating within this repository. 

---

## 1. System Overview & Agent Operating Model

Our repository uses a hybrid microservices framework designed for real-time telemetry processing and predictive analytics. Agents must respect module isolation and follow strictly typed contracts.

```
                  +-----------------------------------+
                  |      Agent Ingestion Engine       |
                  +-----------------+-----------------+
                                    |
            +-----------------------+-----------------------+
            |                                               |
            v                                               v
+-----------------------+                       +-----------------------+
|  Deterministic Tasks  |                       |  Heuristic / LLM Task |
| (Format, Lint, Types) |                       | (Refactor, Docs, Specs|
+-----------+-----------+                       +-----------+-----------+
            |                                               |
            +-----------------------+-----------------------+
                                    |
                                    v
                  +-----------------------------------+
                  |      Verification Pipeline        |
                  |  (Build, Test, Security Analysis) |
                  +-----------------+-----------------+
                                    |
                                    v
                  +-----------------------------------+
                  |      Pull Request / Artifact      |
                  +-----------------------------------+
```

---

## 2. Agent Skill Matrix & Capability Routing

Agents should query their internal skill matrix to ensure they possess the required operational level before attempting automated modifications.

| Skill Identifier | Functional Domain | Risk Level | Validation Required | Primary Output Target |
| :--- | :--- | :---: | :---: | :--- |
| `skill-ts-refactor` | TypeScript Core | Medium | Strict TypeCheck | `src/core/` |
| `skill-sql-migrate` | Database Schema | **High** | Migration Test Suite | `prisma/migrations/` |
| `skill-api-gen` | OpenAPI / REST | Low | Schema Validation | `src/api/generated/` |
| `skill-test-coverage` | Jest / Unit Test | Low | Test Runner | `tests/unit/` |
| `skill-sec-audit` | Dependency & Code Security | **Critical** | SAST Scanner | Security Report |
| `skill-doc-sync` | Docs & Markdown | Low | Linter / Link Check | `docs/`, `*.md` |

---

## 3. Directory Structure & Execution Boundaries

Agents are restricted from modifying core engine logic without explicit high-risk execution flags.

```
.
├── .github/                  # CI/CD Workflows (READ-ONLY for standard agents)
├── config/                   # Global configuration files
│   ├── agent-rules.json      # Machine-readable agent policy definitions
│   └── telemetry.yaml        # System metric specs
├── docs/                     # Architectural specs and documentation
├── src/                      # Application Source Code
│   ├── api/                  # REST and GraphQL Endpoints
│   │   ├── controllers/      # Route handling logic
│   │   └── middleware/       # Auth and logging hooks
│   ├── core/                 # Core engine logic (HIGH-RISK ZONE)
│   │   ├── engine.ts         # Core state machine
│   │   └── stream.ts         # Event stream processing
│   └── utils/                # Pure helper functions
├── tests/                    # Automated Test Suites
│   ├── integration/          # Integration test suites
│   └── unit/                 # High-speed unit tests
├── AGENTS.md                 # Agent governance (This file)
└── CLAUDE.md                 # Claude-specific CLI context shortcuts
```

---

## 4. Operational Boundaries Matrix

| Target Directory / Files | Permitted Actions | Approval Level | Max PR Size | Allowed Automated Fixes |
| :--- | :--- | :---: | :---: | :--- |
| `src/core/**` | Read, Refactor (Scoped) | Staff Eng Review | < 150 Lines | Type fixes, Dead code removal |
| `src/api/**` | Read, Append, Refactor | Peer Review | < 300 Lines | Schema updates, Error handling |
| `src/utils/**` | Full Access | Automated CI | < 500 Lines | Optimization, Complete rewrites |
| `tests/**` | Full Access | Automated CI | < 500 Lines | Test additions, Mock updates |
| `.github/**` | Read Only | Security Lead | 0 Lines | None |
| `prisma/schema.prisma` | Read, Append | Database Admin | < 50 Lines | Format, Index suggestions |

---

## 5. Execution Lifecycle & Verification Workflow

Agents must execute tasks following a strict multi-stage verification workflow. Skipping validation stages will cause immediate CI pipeline rejection.

```
 +------------------+      +------------------+      +------------------+
 |  1. Context Parse| ---> | 2. Code Mutation | ---> | 3. Local Lint    |
 +------------------+      +------------------+      +--------+---------+
                                                              |
                                                              v
 +------------------+      +------------------+      +------------------+
 | 6. PR Generation | <--- | 5. Coverage Check| <--- | 4. Integration   |
 +------------------+      +------------------+      +------------------+
```

### Stage Requirements

1. **Context Parsing:** Read relevant target modules and neighboring type interfaces.
2. **Code Mutation:** Perform minimal necessary changes adhering to defined style guidelines.
3. **Local Lint:** Run localized linting (`npm run lint:fix -- <path>`).
4. **Integration Test Execution:** Verify functional behavior against the test harness.
5. **Coverage Verification:** Ensure code coverage delta does not fall below $0.00\%$.
6. **PR Generation:** Format commit messages according to Conventional Commits standards.

---

## 6. Mathematical & Performance Benchmarks

Agents implementing algorithmic updates in `src/core/` must meet the following computational complexity bounds:

### Latency Budget Formula

Total allowable execution latency ($L_{total}$) for processing batch telemetry is bounded by:

$$L_{total}(n) \le O(n \log n) + \sum_{k=1}^{m} P_k$$

Where:
- $n$ is the number of ingested telemetry payloads.
- $m$ is the active pipeline plugin count.
- $P_k$ represents individual plugin latency overheads where $P_k \le 12\text{ms}$.

### Resource Allocation Matrix

| Operation Type | Max CPU Allocation | Memory Ceiling ($M_{max}$) | Max Execution Time ($T_{max}$) |
| :--- | :---: | :---: | :---: |
| Stream Ingestion | 0.5 Cores | 512 MB | 250 ms |
| Batch Transformation | 2.0 Cores | 2048 MB | 5000 ms |
| Schema Validation | 0.2 Cores | 128 MB | 50 ms |
| Cache Eviction | 0.1 Cores | 64 MB | 10 ms |

---

## 7. Command Reference & CLI Shortcuts

Below are standard CLI workflows agents should run during local iteration:

### Environment Setup & Dependency Checks

```bash
# Verify environment health and runtime version consistency
npm run env:check

# Clean dependencies and rebuild lockfile if drift is detected
npm ci --prefer-offline
```

### Testing & Quality Controls

```bash
# Execute fast-path unit tests only
npm run test:unit -- --bail --findRelatedTests <modified-file>

# Execute full coverage check with HTML reporting
npm run test:coverage -- --coverageReporters="text" --coverageReporters="html"

# Run AST-level static security scanner
npm run audit:sast
```

### Code Formatting & Auto-Fixing

```bash
# Format Markdown, JSON, and TypeScript files
npx prettier --write "src/**/*.{ts,tsx}" "docs/**/*.md"

# Lint TypeScript code with auto-fix enabled
npx eslint "src/**/*.{ts,tsx}" --fix
```

---

## 8. State Transition Rules

Agents managing background asynchronous state engines must conform to the following deterministic state transitions:

```
                  +-----------------------+
                  |         IDLE          |
                  +-----------+-----------+
                              |
                              | Task Enqueued
                              v
                  +-----------------------+
                  |       INITIALIZED     |
                  +-----------+-----------+
                              |
               +--------------+--------------+
               |                             |
     Resource Available             Resource Exhausted
               |                             |
               v                             v
    +-------------------+          +-------------------+
    |    PROCESSING     |          |      QUEUED       |
    +---------+---------+          +---------+---------+
              |                              |
      +-------+-------+                      | Timeout Exceeded
      |               |                      |
   Success          Failure                  v
      |               |            +-------------------+
      v               v            |      FAILED       |
+-----------+   +-----------+      +-------------------+
| COMPLETED |   | RETRYING  |                ^
+-----------+   +-----+-----+                |
                      |                      |
                      +-- Max Retries Exceeded
```

---

## 9. Failure Recovery Protocols

When an automated process fails, the handling agent must follow standard escalation protocols:

```
[Agent Detects Error]
         |
         +--> Is Error Type == Syntax/Type check?
         |          |
         |          +--> YES: Attempt auto-fix (Max 3 attempts).
         |          +--> NO: Proceed to test failure check.
         |
         +--> Is Error Type == Test Failure?
         |          |
         |          +--> YES: Revert last diff, inspect mock harness.
         |          +--> NO: Proceed to environment check.
         |
         +--> Is Error Type == Infrastructure / Timeout?
                    |
                    +--> YES: Abort execution, post log summary to PR comments.
```

---

## 10. Memory & Staleness Management

Agents must maintain documentation freshness and state alignment.

```
       Last Commits to /src
                 |
                 v
    +-------------------------+
    | Calculate Git Hash Delta|
    +------------+------------+
                 |
                 v
    Is Delta > 15 Commits or > 14 Days?
                 |
        +--------+--------+
        |                 |
       YES                NO
        |                 |
        v                 v
+---------------+  +---------------+
| Flag Doc as   |  | Doc Status:   |
| "STALE"       |  | "FRESH"       |
+-------+-------+  +---------------+
        |
        v
+-----------------------------------+
| Trigger "skill-doc-sync" Agent    |
+-----------------------------------+
```

### Staleness Resolution Protocol

1. **Detection:** When `AGENTS.md` or `CLAUDE.md` drifts from codebase state by more than **15 commits**, mark document status as `STALE`.
2. **Analysis:** Analyze AST changes in core exports (`src/core/`, `src/api/`).
3. **Diff Proposal:** Generate suggested updates via non-destructive PR annotations.
4. **Human Verification:** Require approval from code owners before updating governance docs.

---

## 11. Security & Compliance Checklist

Agents must complete this automated pre-flight checklist before submitting code modifications.

- [ ] **No Hardcoded Secrets:** Check code for API keys, bearer tokens, or password strings.
- [ ] **Type Soundness:** Validate that no `any` types were introduced without an explicit inline comment explaining why.
- [ ] **Input Sanitization:** Ensure all external HTTP inputs pass through a validation schema.
- [ ] **Dependency Audit:** Verify no new `npm` packages were added without vulnerability scanning.
- [ ] **Log Privacy:** Verify that no personally identifiable information (PII) is emitted to stdout/stderr.

---

## 12. Document Governance & Maintenance

| Event | Action Required | Responsible Agent / Role |
| :--- | :--- | :--- |
| API Route Addition | Update `src/api/generated/` and OpenAPI spec | `skill-api-gen` |
| Database Migration | Regenerate Prisma client and test suites | `skill-sql-migrate` |
| Skill Capability Drift | Run update check on skill definitions | `skill-doc-sync` |
| Security Advisory | Block CI run and generate patch report | `skill-sec-audit` |

*For queries regarding agent configuration, consult the repository maintainers or raise an issue with the label `agent-governance`.*
