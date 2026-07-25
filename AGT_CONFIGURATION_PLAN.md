# AGT configuration plan

Status: decisions recorded; implementation plan ready  
Date: 2026-07-25  
Current deployment: official AGT repository checkout with the Streamlit
dashboard running in Docker at `http://127.0.0.1:18501`.

This plan separates the dashboard container from the enforcement integrations.
The dashboard is useful for visibility, but agents are governed only when the
appropriate AGT SDK, plugin, middleware, or sidecar is connected to their tool
and message paths.

## How to answer

Reply with one choice per number, for example:

```text
1A, 2A, 3B, 4A, 5A, 6A, 7A, 8A, 9A, 10A,
11A, 12A, 13B, 14A, 15A, 16A, 17A, 18A, 19A, 20A
```

You may add a short note after any answer. If you do not know, select the
recommended option and I will produce a conservative first configuration.

## Recorded decisions

The user selected the recommended **A** option for all 20 questions:

```text
1A, 2A, 3A, 4A, 5A, 6A, 7A, 8A, 9A, 10A,
11A, 12A, 13A, 14A, 15A, 16A, 17A, 18A, 19A, 20A
```

### Resulting baseline

- Govern Codex, OpenCode, Claude Code, Gemini CLI, and Hermes.
- Use the container for dashboard plus policy/audit support.
- Begin in evaluate-only mode, then promote to enforcement.
- Use the native AGT YAML/ACS policy engine.
- Cover all available model, prompt, tool, delegation, and output
  intervention points.
- Default to least privilege: unmatched side-effecting actions deny; risky
  actions require approval or denial.
- Use stable per-agent identities, runtime secret injection, local append-only
  audit storage, payload-minimized telemetry, loopback dashboard access,
  hardened containers, pinned monthly updates, Git-reviewed policies, full
  negative testing, staged rollout, and one-command rollback.

Implementation should proceed in evaluate-only mode first. Selecting these
defaults does not authorize switching live agents to enforce mode without a
separate smoke test and rollout review.

## Questions

### 1. Which agent surfaces should AGT govern first?

- **A — Codex, OpenCode, Claude Code, Gemini CLI, and Hermes (Recommended):** use the shared agent-context layer and cover the tools already present on this host.
- B — OpenCode and Claude Code only: smaller initial scope.
- C — One application agent only: defer CLI-wide governance.

### 2. What is the primary operating goal?

- **A — Local personal-agent safety and auditability (Recommended):** protect this host while keeping the design extensible.
- B — Production multi-user governance: design for teams, central identity, and formal operations immediately.
- C — Evaluation/research: prioritize experiments and metrics over blocking.

### 3. Which AGT deployment role should the container have?

- **A — Dashboard plus policy/audit support (Recommended):** retain the current dashboard and add only the services required by selected integrations.
- B — Dashboard only: enforcement remains entirely in-process in each agent.
- C — Dedicated policy decision point: build a separately addressable enforcement service.

### 4. What should the initial policy mode be?

- **A — Evaluate-only for a bake-in period, then enforce (Recommended):** observe decisions before blocking real work.
- B — Enforce immediately: block policy violations from the first run.
- C — Advisory only: log findings but never block.

### 5. Which policy backend should be authoritative?

- **A — AGT native YAML/ACS policy engine (Recommended):** use the toolkit’s deterministic policy path and version policies in Git.
- B — OPA/Rego: use an existing OPA policy estate.
- C — Cedar/Cedarling: use Cedar as the organization-wide authorization language.

### 6. Which intervention points must be governed?

- **A — All available points: input, model request/response, tool call/result, delegation, and final output (Recommended):** maximize mediation coverage.
- B — Tool calls and tool results only: focus on the highest-risk side effects.
- C — Prompts and final output only: begin with content safety.

### 7. What should happen when no rule matches?

- **A — Deny for side-effecting actions and allow read-only actions (Recommended):** use explicit action classes with least privilege.
- B — Allow: preserve compatibility and add rules incrementally.
- C — Require review: send all unmatched actions to an approval path.

### 8. Which actions must always be blocked without approval?

- **A — Destructive filesystem, shell, database, credential, network, and message-send actions (Recommended):** deny high-impact operations by default.
- B — Only destructive filesystem and shell actions: narrower initial policy.
- C — No unconditional blocks: rely on human review for all risky actions.

### 9. How should AGT handle review/escalate decisions?

- **A — Use the host agent’s normal permission/approval flow and fail closed if unavailable (Recommended):** avoid inventing a second approval system.
- B — Manual operator approval through a future dashboard workflow.
- C — Convert every review to deny until an approval service is built.

### 10. What identity model should govern agents?

- **A — Stable per-agent identity names now; workload identity later (Recommended):** establish audit attribution without adding cloud dependencies.
- B — SPIFFE/SVID or managed workload identity immediately.
- C — User identity only: do not distinguish individual agents yet.

### 11. Where should AGT secrets and signing material live?

- **A — Existing Proton Pass/secret-source workflow; inject at runtime only (Recommended):** keep secrets out of policies, images, and Git.
- B — Docker Compose `.env` file with mode `0600`.
- C — Azure Key Vault/managed identity: use a cloud secret manager now.

### 12. Where should audit records be stored initially?

- **A — Host-local append-only, access-controlled AGT audit directory with rotation (Recommended):** simple and private for this host.
- B — Existing OpenLIT/OTel stack plus a local integrity chain.
- C — Azure immutable Blob Storage/WORM: centralize records immediately.

### 13. What retention policy should apply to local audit data?

- **A — 90 days operationally, with manual incident hold (Recommended):** limit local growth while retaining useful investigation history.
- B — One year: prioritize longer local history.
- C — Three years or more: meet formal compliance retention requirements.

### 14. What telemetry should be enabled?

- **A — Decision metadata, latency, allow/warn/deny/escalate counts, and error reasons; exclude payloads (Recommended):** useful observability with data minimization.
- B — Full prompts, tool arguments, and results for debugging.
- C — Minimal counters only: no detailed decision telemetry.

### 15. How should the dashboard be reachable?

- **A — Loopback only, with Tailscale or an authenticated reverse proxy for remote access (Recommended):** preserve the current safe default.
- B — Tailscale interface directly.
- C — LAN-wide or public bind: expose it on `0.0.0.0`.

### 16. What container hardening level is required?

- **A — Non-root user, read-only root filesystem where compatible, resource limits, dropped capabilities, and no host socket (Recommended):** harden the dashboard without enabling arbitrary host control.
- B — Current upstream developer container defaults.
- C — Privileged container with host integration: required only if a specific feature proves it necessary.

### 17. How should toolkit updates be managed?

- **A — Pin Git commit/image dependencies, review changelog, rebuild and smoke-test monthly (Recommended):** balance reproducibility and maintenance.
- B — Track the moving `main` branch and rebuild weekly.
- C — Pin a release indefinitely and update only after incidents.

### 18. Who may change policies?

- **A — Git-reviewed changes by the user, with AGT lint/tests required (Recommended):** keep policy changes auditable.
- B — Agents may propose changes, but a human must approve and commit them.
- C — Agents may edit and activate policies directly.

### 19. What validation gate should be required before enforcement?

- **A — Policy lint, unit tests, negative destructive-action tests, prompt-injection tests, and a shadow/evaluate-only report (Recommended):** prove both intended allows and blocks.
- B — Policy lint and a basic smoke test.
- C — Manual review only.

### 20. How should rollout and rollback work?

- **A — Per-agent staged rollout, versioned policy bundle, health checks, and one-command rollback (Recommended):** reduce blast radius.
- B — Activate one global policy for all agents at once.
- C — Manual edits in each agent configuration.

## Proposed implementation phases after answers

1. Freeze the selected scope and threat model.
2. Create versioned policies and a policy-test fixture set under the canonical
   agent-context repository or the AGT checkout, without secrets.
3. Configure OpenCode and Claude Code integrations, then add adapters for the
   remaining selected agents.
4. Connect decision metadata to the chosen audit/telemetry sink.
5. Run evaluate-only against representative workflows and review false
   positives, bypasses, latency, and missing intervention points.
6. Enable enforce mode for low-risk agents first, then promote gradually.
7. Verify rollback, audit integrity, container restart behavior, and policy
   update procedures.

## Guardrails already selected by this plan

- The dashboard stays bound to `127.0.0.1:18501` unless explicitly changed.
- No secrets are written into this plan, policies, Dockerfiles, or Git.
- The official toolkit source is retained at this checkout and the container is
  rebuilt from the pinned checkout state.
- Runtime policy errors should fail closed for enforce-mode integrations.
- Audit data should exclude raw prompts, tool arguments, tool results, and
  personal data unless a documented exception is approved.

## References

- [Official Agent Governance Toolkit repository](https://github.com/microsoft/agent-governance-toolkit)
- [Toolkit deployment guides](docs/deployment/README.md)
- [Production deployment guidance](policy-engine/docs/PRODUCTION_DEPLOYMENT.md)
- [Security hardening tutorial](docs/tutorials/25-security-hardening.md)
- [OpenCode integration](agent-governance-opencode/README.md)
- [Record retention policy](docs/compliance/record-retention-policy.md)

## Implementation result (2026-07-25)

- Central policy source: `/home/server/data/repos/agent-context/policies`.
- Active profiles: `opencode.advisory.json` and `claude.advisory.json`, reached
  through runtime symlinks; enforce profiles remain preserved for promotion.
- Native candidate: `agt-baseline-enforce-candidate.yaml`; `agt lint-policy`
  passes.
- Audit location: `/home/server/data/repos/agent-context/audit`, mode `0700`,
  Git-ignored, with payload minimization retained.
- Dashboard: healthy at `http://127.0.0.1:18501`, with loopback binding,
  `restart: unless-stopped`, read-only rootfs, dropped capabilities,
  `no-new-privileges`, tmpfs write areas, PID/CPU/memory limits, and a health
  check.
- OpenCode and Claude package tests completed successfully (the installed
  packages currently contain no test cases, so the runner reported zero
  tests).
- Codex, Gemini CLI, and Hermes have central guidance paths but no equivalent
  first-party AGT pre-tool enforcement hook in the installed toolkit release.
  Do not promote enforcement for those surfaces based on this setup alone.
- The user-systemd unit is installed, but this execution environment cannot
  connect to the user systemd bus; Docker restart policy is the effective
  automatic-restart mechanism until the unit is enabled in a normal login
  session.
