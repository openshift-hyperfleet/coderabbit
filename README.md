# CodeRabbit Central Configuration

Central CodeRabbit configuration for the `openshift-hyperfleet` GitHub organization.

The `.coderabbit.yaml` in this repository automatically applies to all repos in the org that don't have their own `.coderabbit.yaml`.

## Configuration hierarchy

CodeRabbit uses the following priority order (highest to lowest):

1. **Global Overrides** — Red Hat enterprise-level settings (managed by PTLT)
2. **Repository `.coderabbit.yaml`** — per-repo overrides
3. **Central repository `.coderabbit.yaml`** — this file (applies to all repos)
4. **Organization Settings** — CodeRabbit web UI for the org
5. **Default Settings** — CodeRabbit built-in defaults

With `inheritance: true`, repo-level configs **merge** with this central config instead of replacing it entirely.

## What this configures

| Area | Configuration |
|------|--------------|
| Review profile | `chill` — focused on critical issues, minimal noise |
| Path instructions | Go mechanical checks (error handling, concurrency, exhaustiveness, resource lifecycle, code quality, naming, performance), language-agnostic checks (security, code hygiene), testing standard, plus stricter review for `cmd/`, `config/`, `deploy/`, `charts/`, `migrations/` |
| Code guidelines | Reads `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/*.mdc` from each repo |
| Linked repositories | Architecture, API, Sentinel, Adapter, Broker — cross-repo analysis |
| Tools | `golangci-lint`, `gitleaks`, `yamllint`, `markdownlint` enabled |
| Auto-review | Enabled, skips drafts and WIP PRs |

## Adding repo-specific overrides

Create a `.coderabbit.yaml` in your repo root. With `inheritance: true`, your settings merge with the central config:

```yaml
# yaml-language-server: $schema=https://coderabbit.ai/integrations/schema.v2.json
inheritance: true

reviews:
  profile: chill  # Override to lighter feedback for this repo

  path_instructions:
    - path: "internal/special/**"
      instructions: |
        Additional repo-specific review instructions.
```

> **Warning:** `inheritance: true` is **required** in every per-repo `.coderabbit.yaml`. Without it, CodeRabbit silently ignores the entire central config — all path instructions, review profile, and tool settings disappear for that repo. Additionally, `path_instructions` arrays dedupe by path key: if your repo defines `**/*.go`, it shadows the central `**/*.go` entry entirely (instructions are NOT concatenated). Do NOT redefine `**`, `**/*.go`, or `**/*_test.go` path keys unless you intentionally want to replace the central code review checks.

See the [CodeRabbit configuration docs](https://docs.coderabbit.ai/guides/configuration-overview) for all available options.

## Pre-commit security policy

The central rule follows the [HyperFleet hook guide](https://github.com/openshift-hyperfleet/architecture/blob/main/hyperfleet/docs/pre-commit-hooks.md). It recognizes `github.com/leaktk/leaktk` and `github.com/openshift-hyperfleet/hyperfleet-hooks` as exact sources, not their entire organizations. The existing trusted-owner list is otherwise unchanged.

All remote hooks require full 40-character Git SHAs and matching `# frozen: <tag>` comments, including hooks from trusted sources. Verified annotated-tag object SHAs are valid frozen refs; they need not equal the peeled commit SHA. `repo: local` and `repo: meta` do not have remote revisions. Source trust and SHA pinning are not proof of safe code or execution isolation.

The only system-hook exception is the published HyperFleet registry's `hyperfleet-gofmt` → `make gofmt`, `hyperfleet-golangci-lint` → `make lint`, and `hyperfleet-go-vet` → `make go-vet`. Review the pinned manifest, consumer overrides/arguments, and any changes to the delegated Make recipes and their dependencies. The exception does not approve new revisions automatically, allow arbitrary commands, or waive human review and CI. There is no corresponding blanket exception for local or script hooks.

### Review regression cases

These are expected review outcomes, not an executable security scanner. YAML/schema validation can check configuration structure, but cannot prove that the reviewer follows natural-language instructions.

| Case | Expected outcome |
|------|------------------|
| LeakTK's documented Go hook, exact source, verified full SHA and frozen release comment | No generic untrusted-source finding; review revision changes normally |
| HyperFleet commitlint, exact source, verified frozen SHA, published Go entrypoint | No generic untrusted-source finding; no system-hook exception needed |
| Each of the three documented HyperFleet system hooks with its exact published entrypoint and unchanged safe Make recipe | No finding solely for source trust or `language: system` |
| Verified frozen annotated-tag object resolving to the stated release | Accept as a full Git SHA; do not demand the peeled commit instead |
| Another repo under `leaktk` or `openshift-hyperfleet`, or a lookalike owner/host/repository | No new source exception; flag an untrusted origin |
| `rev: main`, a release tag, or an abbreviated SHA, even with a frozen comment or trusted source | Critical immutable-pinning finding |
| Full SHA with a missing or mismatched frozen comment | Update-metadata finding; not by itself evidence of malicious code |
| A new system hook ID, changed entrypoint, or consumer arguments invoking additional commands/targets | Outside the exception; flag unexpected execution |
| A local system hook reusing one of the three HyperFleet IDs | Not covered by the remote-source exception; review command execution |
| A documented target or its transitive recipe/script changed to execute unverified downloads, upload credentials, or interpolate untrusted shell input | Security finding remains required, regardless of hook source |
| Manifest unavailable at the pinned SHA | Report an explicit verification gap; do not assume the hook matches |
| Local/meta hook with no `rev` | Do not request a remote SHA; other execution checks still apply |

### Validation and rollout

1. Parse `.coderabbit.yaml` and validate it against the [CodeRabbit v2 schema](https://coderabbit.ai/integrations/schema.v2.json).
2. Compare the parsed config with the base branch: changes should be limited to the pre-commit and Makefile path instructions, preserving the other review settings.
3. Review the cases above with the central policy maintainers. This repository has no automated integration/E2E harness for CodeRabbit review behavior.
4. After the central config is merged and active, request a fresh review of a representative adoption/migration PR and verify the positive cases no longer produce the generic trust/system findings. Verify negative cases in controlled review fixtures before treating rollout as complete. Do not weaken enterprise overrides or add consumer-local overrides to bypass them.

## Current limitations

### Learnable rules disabled

`knowledge_base.opt_out: true` is mandatory across all Red Hat CodeRabbit repos due to contractual data retention restrictions. This disables learnable rules (rules that improve from reviewer feedback over time). Under review by the PTLT team — no timeline for resolution.

### Web search disabled

`web_search.enabled: false` is set at the Red Hat enterprise level.

### Code guidelines are repo-local only

`code_guidelines.filePatterns` only matches files within the reviewed repository. Cross-repo guideline files (e.g. from the architecture repo) cannot be referenced here. The mechanical code review checks from the architecture repo are inlined as condensed summaries in `path_instructions` instead. See the YAML comments in `.coderabbit.yaml` for details.

### JIRA integration not approved

The CodeRabbit JIRA integration is not currently approved due to security concerns raised by PTLT. This may be revisited in the future. For JIRA ticket validation during PR reviews, use the Claude Code `/review-pr` skill instead.

## TODO: manual setup steps

### CodeRabbit app permissions

Ensure the CodeRabbit GitHub app is installed on this config repository and all linked repositories:

- `openshift-hyperfleet/architecture`
- `openshift-hyperfleet/hyperfleet-api`
- `openshift-hyperfleet/hyperfleet-sentinel`
- `openshift-hyperfleet/hyperfleet-adapter`
- `openshift-hyperfleet/hyperfleet-broker`

The app is likely already installed if CodeRabbit is reviewing PRs in these repos.

## Reference

- [CodeRabbit configuration overview](https://docs.coderabbit.ai/guides/configuration-overview)
- [Central configuration docs](https://docs.coderabbit.ai/configuration/central-configuration)
- [Multi-repo analysis](https://docs.coderabbit.ai/knowledge-base/multi-repo-analysis)
- [Tool integrations](https://docs.coderabbit.ai/tools/list)
- [HyperFleet automated PR review strategy](https://github.com/openshift-hyperfleet/architecture/blob/main/hyperfleet/docs/automated-pr-review-strategy.md)
