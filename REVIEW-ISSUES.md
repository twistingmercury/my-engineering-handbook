# Engineering Handbook Review Issues

> **Generated**: 2026-01-30
> **Status**: In Progress
> **Total Issues**: 47

Track progress by checking off items as they are resolved.

---

## Critical Cross-Document Issues

These require coordination across multiple documents.

- [√] **CRIT-01**: Emoji rule violation
  - `importance-of-documentation.md` states "No emojis"
  - `builds-and-deployments.md` line 22 contains: "you get the idea :smile:"
  - **Action**: Remove emoji OR revise the rule

- [√] **CRIT-02**: E2E testing guidance fragmentation
  - `builds-and-deployments.md` covers infrastructure/Docker setup
  - `deliver-solutions-that-work.md` covers testing philosophy
  - No cross-references between them
  - **Action**: Add cross-references or consolidate

- [√] **CRIT-03**: Health check endpoint design conflict
  - `observability-heartbeats-readiness-liveness.md` provides single `/ops/health` endpoint
  - Title implies separate readiness and liveness (Kubernetes best practice)
  - **Action**: Decide on unified vs separate endpoints; update doc accordingly

- [√] **CRIT-04**: Status enum inconsistency
  - `HealthResponse` uses: `[OK, WARNING, ERROR]`
  - `DependencyHealth` uses: `[OK, WARNING, CRITICAL]`
  - **Action**: Standardize on ERROR or CRITICAL across both schemas

- [√] **CRIT-05**: Correlation ID standard missing
  - `observability-distributed-tracing.md` references correlation IDs
  - `observability-logging.md` references correlation IDs
  - Neither specifies generation, propagation, or format
  - **Action**: Define correlation ID standard in one doc, reference from other

- [√] **CRIT-06**: Contradictory monitoring guidance
  - `scale-and-high-availability.md`: "Monitor everything"
  - `observability-metrics.md`: warns against over-instrumenting
  - **Action**: Reconcile guidance; clarify what "everything" means

---

## Ambiguous Thresholds & Terms

These use vague language that needs quantification or clarification.

### Managing Our Source

- [√] **AMB-01**: "Short-lived" branches undefined
  - **Current**: No timeframe specified
  - **Action**: Define expected duration (e.g., "less than 1 week")
  - **Recommendation**: 4 working days maximum. PRs from branches older than 3 days require justification in the PR description.

- [√] **AMB-02**: "Regularly merge from main" undefined
  - **Current**: No frequency specified
  - **Action**: Define frequency (e.g., "at least daily")
  - **Recommendation**: At least once daily, and always before requesting review

### Deliver Solutions That Work

- [√] **AMB-03**: "Good coverage" undefined
  - **Current**: States "not 100%" but no target
  - **Action**: Define acceptable range or criteria
  - **Recommendation**: 75% minimum overall, 95% for critical paths (document exceptions with justification)

- [√] **AMB-04**: "Once confident" for auto-deploy undefined
  - **Current**: "history of deployments going well"
  - **Action**: Define criteria (e.g., "10 consecutive successful deployments")
  - **Recommendation**: 20 consecutive successful deployments over minimum 30 days with zero rollbacks

### Flexible Application Configuration

- [√] **AMB-05**: "Safe values" for development undefined
  - **Current**: Only "localhost" mentioned
  - **Action**: Provide more examples or guidelines
  - **Recommendation**: Add examples: localhost URLs, conservative timeouts (30s), feature flags OFF. Principle: defaults should fail loudly in production.

- [√] **AMB-06**: "Fail fast" behavior undefined
  - **Current**: No guidance on implementation
  - **Action**: Define expected behavior (exit code, logging, retry policy)
  - **Recommendation**: Exit code 1, log at CRITICAL level with specific field names, validate all config before exiting (not just first failure), no retry loops

### Observability: Distributed Tracing

- [√] **AMB-07**: Sampling rate adjustment criteria missing
  - **Current**: "Start with 10%" with no adjustment guidance
  - **Action**: Define when/how to adjust (traffic thresholds, cost targets)
  - **Recommendation**: 10% default (<1000 req/min), 1-5% (1000-10000 req/min), 0.1-1% (>10000 req/min). Always 100% for errors. Review monthly; after a release, review daily for 5 days.

- [√] **AMB-08**: "Expensive operations" undefined
  - **Current**: No threshold for what's expensive
  - **Action**: Define threshold (e.g., ">100ms" or ">1s")
  - **Recommendation**: Tiered thresholds by operation type: User-facing API >200ms (Google SRE), Database >100ms (PostgreSQL/MySQL), Cache >10ms (Redis docs), External API >500ms (AWS), Memory >50MB, All errors always. Include source references.

### Observability: Logging

- [√] **AMB-09**: "Sanitized stack traces" undefined
  - **Current**: No sanitization guidance
  - **Action**: Define what to remove (file paths, usernames, etc.)
  - **Recommendation**: Remove absolute paths (show relative), usernames, home directories, env variable values, memory addresses. Keep function names, line numbers. Truncate to 50 frames.

- [√] **AMB-10**: "Approaching resource limits" undefined
  - **Current**: No percentage threshold
  - **Action**: Define threshold (e.g., "80% utilization")
  - **Recommendation**: 75% = WARN, 80% = ERROR, 90% = CRITICAL. Log immediately at these levels for forensics; only alert/page if sustained 5+ minutes.

- [√] **AMB-10a**: Daily log review not documented as operational task
  - **Current**: No operational cadence for log review
  - **Action**: Add operational requirement for log review
  - **Recommendation**: Add to logging doc: "On-call engineer reviews logs daily as part of operational duties"

### Scale & High Availability

- [√] **AMB-11**: Caching threshold vague
  - **Current**: "few hundred calls a day"
  - **Action**: Define specific threshold (e.g., "<500 requests/day")
  - **Recommendation**: Cache when >500 req/day AND (expected hit ratio >60% OR origin latency >100ms OR approaching 70% of rate limits). Document invalidation strategy.

- [√] **AMB-12**: "Test scaling triggers regularly" undefined
  - **Current**: No frequency specified
  - **Action**: Define frequency (e.g., "monthly" or "quarterly")
  - **Recommendation**: Monthly for production systems, quarterly for non-critical. Test both scale-up and scale-down.

### Security in Our Development Process

- [√] **AMB-13**: "Rotate secrets regularly" undefined
  - **Current**: No timeframe
  - **Action**: Define frequency (e.g., "every 90 days")
  - **Recommendation**: 90 days default, 30 days for DB passwords/external-facing keys, immediate on team departure or suspected compromise. Automate via secrets manager.

### Versioning Our Solutions

- [√] **AMB-14**: "Production-ready" undefined
  - **Current**: No definition
  - **Action**: Define criteria or link to Definition of Done
  - **Recommendation**: Link to Definition of Done. Explicit criteria: CI green, code review approved, observability configured, runbook created, staging tested, rollback documented.

---

## Unacknowledged Assumptions

These assume tools/practices without explicit documentation.

### Tooling Assumptions (Need Central Documentation)

- [ ] **ASM-01**: Docker/Kubernetes assumed everywhere
  - **Mentioned in**: All 12 documents
  - **Action**: Create tooling standards doc OR add prerequisites section
  - **Recommendation**: Create central "Tooling Standards" document stating Docker/Kubernetes as foundational platform

- [ ] **ASM-02**: Prometheus assumed for metrics
  - **Mentioned in**: observability-metrics.md
  - **Action**: Explicitly state as standard or document alternatives
  - **Recommendation**: Make explicit in observability-metrics.md: "We use Prometheus-compatible metrics format"

- [ ] **ASM-03**: Datadog assumed for observability
  - **Mentioned in**: deliver-solutions-that-work.md
  - **Action**: Clarify if required or example
  - **Recommendation**: Change to example, not requirement: "observability platform (e.g., Datadog)"

- [ ] **ASM-04**: Confluence assumed for wiki
  - **Mentioned in**: importance-of-documentation.md
  - **Action**: Explicitly state as standard
  - **Recommendation**: State explicitly: "Team wiki is hosted in Confluence"

- [ ] **ASM-05**: Jira assumed for ticket tracking
  - **Mentioned in**: importance-of-documentation.md, managing-our-source.md
  - **Action**: Explicitly state as standard
  - **Recommendation**: State explicitly: "We use Jira for issue tracking. Branch names include Jira ticket numbers."

- [ ] **ASM-06**: Azure vs AWS unclear
  - **Azure in**: builds-and-deployments.md, scale-and-high-availability.md
  - **AWS in**: flexible-application-configuration.md, security-in-development.md
  - **Action**: Clarify if multi-cloud or pick primary
  - **Recommendation**: Clarify multi-cloud in Tooling Standards: "Primarily Azure (AKS), some AWS services. Prefer cloud-agnostic examples."

- [ ] **ASM-07**: Redis assumed for caching/sessions
  - **Mentioned in**: scale-and-high-availability.md
  - **Action**: Explicitly state as standard or document alternatives
  - **Recommendation**: State as standard: "Redis is our standard for distributed caching and session storage"

### Architectural Assumptions

- [ ] **ASM-08**: Microservices architecture assumed
  - **Action**: State explicitly or acknowledge monolith scenarios
  - **Recommendation**: Add note: "All services must be containerized and stateless regardless of architecture pattern"

- [ ] **ASM-09**: PR-based workflow assumed
  - **Action**: State explicitly in managing-our-source.md
  - **Recommendation**: Add: "All changes require a pull request. Direct commits to main are blocked."

- [ ] **ASM-10**: Dedicated Security/Compliance teams assumed
  - **Mentioned in**: security-in-development.md
  - **Action**: Clarify escalation for teams without dedicated security
  - **Recommendation**: Add escalation guidance for teams without dedicated security: escalate to team lead, use automated scanning, monthly security reviews

---

## Document-Specific Issues

### builds-and-deployments.md

- [ ] **DOC-01**: Dockerfile section describes orchestrator script steps
  - Lists 4 CI build parts but section mixes responsibilities
  - **Action**: Clarify which steps belong to Dockerfile vs build.sh
  - **Recommendation**: Restructure "Basic Build Architecture" - separate Dockerfile responsibilities from build.sh orchestrator steps, add table showing which component does what

- [ ] **DOC-02**: Internal project name leaked
  - References "SAMPLE MGMT API" without context
  - **Action**: Generalize or remove specific project references
  - **Recommendation**: Replace "SAMPLE MGMT API" with generic "example-api" or "your API"

### deliver-solutions-that-work.md

- [ ] **DOC-03**: Unit test file system guidance too restrictive
  - Says no "file system" but file parsing is legitimate
  - **Action**: Clarify: no external file dependencies, but local test fixtures OK
  - **Recommendation**: Change to: "no external file system dependencies. Local test fixtures and embedded test data are acceptable."

- [ ] **DOC-04**: Tension between exhaustive and pragmatic testing
  - "Test every endpoint" vs "use your judgment"
  - **Action**: Clarify priority or provide decision framework
  - **Recommendation**: Add prioritization: 1) All success paths, 2) Documented error responses, 3) Auth boundaries, 4) Edge cases. Use judgment on permutations.

### flexible-application-configuration.md

- [ ] **DOC-05**: Section header typo
  - "Configuring Importance" should be "Configuration Importance"
  - **Action**: Fix typo
  - **Recommendation**: Fix typo: "Configuring Importance" → "Why Configuration Matters"

- [ ] **DOC-06**: Config files vs secrets guidance unclear
  - Says config files are "version controllable" but "never put secrets in config"
  - **Action**: Clarify what goes where
  - **Recommendation**: Add table: Config files (non-sensitive), ConfigMaps (environment-specific), Secrets Manager (credentials)

### importance-of-documentation.md

- [ ] **DOC-07**: Maturity levels undefined
  - `[Emerging|Basic|Mature]` mentioned but not defined
  - **Action**: Add definitions for each level
  - **Recommendation**: Define maturity levels: Emerging (prototype), Basic (production-ready, evolving), Mature (stable, battle-tested)

- [ ] **DOC-08**: "OpsBook" undefined
  - Term used but never explained
  - **Action**: Define or link to explanation
  - **Recommendation**: Define on first use: "OpsBook (operational runbook in Confluence for maintenance procedures)"

- [ ] **DOC-09**: CHANGELOG exception unclear
  - "No exceptions" for README but CHANGELOG only for "versioned releases"
  - **Action**: Clarify what projects don't have versioned releases
  - **Recommendation**: Clarify: Required for versioned releases. Internal tools without version dependencies may use commit history instead.

### managing-our-source.md

- [ ] **DOC-10**: Date discrepancy in metadata
  - Revision history: 2025-10-06
  - Document date field: 2024-11-11
  - **Action**: Correct the dates
  - **Recommendation**: Fix dates: document date should be 2025-10-06 to match revision history

### observability-logging.md

- [ ] **DOC-11**: User ID as PII ambiguous
  - Example shows `user_id` as good practice
  - Also says never log "personal information"
  - **Action**: Clarify if user_id is considered PII
  - **Recommendation**: Clarify: Internal user IDs (UUIDs) are acceptable. Never log the personal information those IDs reference.

- [ ] **DOC-12**: Log level inconsistency in example
  - "Login failure" example uses `info` level
  - Should be `warn` per level definitions
  - **Action**: Fix example to use correct level
  - **Recommendation**: Fix example: change log level from "info" to "warn" for login failure

### security-in-development.md

- [ ] **DOC-13**: Mixed message on manual review
  - "Our CI does this automatically, but look anyway"
  - **Action**: Clarify expectation: required or recommended?
  - **Recommendation**: Clarify: CI runs scans automatically. Developers verify in PR checks: actively maintained, security track record, compatible license.

- [ ] **DOC-14**: Scanner behavior inconsistent
  - Uses "should run" (optional?) and "build fails" (mandatory?)
  - **Action**: Clarify: is security scanning mandatory?
  - **Recommendation**: Make mandatory: "Security scans are mandatory. Builds fail on critical/high findings. Medium findings must be addressed within sprint."

---

## Missing Cross-References

- [ ] **XREF-01**: Definition of Done should link to README requirements
  - From: deliver-solutions-that-work.md
  - To: importance-of-documentation.md
  - **Recommendation**: deliver-solutions-that-work.md → Add link after "README.md is current": "See [README Requirements](./importance-of-documentation.md#project-readme-requirements)"

- [ ] **XREF-02**: E2E testing sections should cross-reference
  - Between: builds-and-deployments.md and deliver-solutions-that-work.md
  - **Recommendation**: Already resolved in CRIT-02. Verify bidirectional links exist.

- [ ] **XREF-03**: Docker security should reference Docker labeling
  - From: security-in-development.md
  - To: versioning-our-solutions.md
  - **Recommendation**: security-in-development.md → Add in "Lock Down Containers": link to versioning-our-solutions.md#docker-image-labeling

- [ ] **XREF-04**: Logging restrictions should be in observability doc
  - From: security-in-development.md (no PII/PHI rules)
  - To: observability-logging.md
  - **Recommendation**: observability-logging.md → Add in "Never Include" section: link to security-in-development.md for PII/PHI guidelines

- [ ] **XREF-05**: Health endpoint should have single source of truth
  - Currently in: observability-heartbeats-readiness-liveness.md, scale-and-high-availability.md
  - **Action**: Pick canonical location, reference from other
  - **Recommendation**: scale-and-high-availability.md → Replace health endpoint details with link to observability-heartbeats-readiness-liveness.md as canonical source
