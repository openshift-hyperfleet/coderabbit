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

The central rule follows the [HyperFleet hook guide](https://github.com/openshift-hyperfleet/architecture/blob/main/hyperfleet/docs/pre-commit-hooks.md). All sources are now exact repositories, not owner prefixes:

- `github.com/leaktk/leaktk`
- `github.com/openshift-hyperfleet/hyperfleet-hooks`
- `github.com/pre-commit/pre-commit-hooks`
- `github.com/pre-commit/mirrors-mypy`
- `github.com/shellcheck-py/shellcheck-py`
- `github.com/astral-sh/ruff-pre-commit`

The list retains the documented HyperFleet sources and the previously trusted repositories found in an inventory of active organization repos' default-branch root configs on 2026-10-21. The old broad owner grants for `golangci`, `rhysd`, `google`, `gitleaks`, `norwoodj`, `igorshubovych`, `dnephin`, `doublify`, and `adrienverge` had no uses in that inventory and are removed, not expanded into guessed repository names. Existing unlisted sources in legacy configs are not newly approved by this change. Other repositories under any listed owner require a policy PR.

All remote hooks require full 40-character Git SHAs and matching `# frozen: <tag>` comments, including hooks from trusted sources. The recorded refs (LeakTK v0.3.12, HyperFleet hooks v0.3.0, and pre-commit-hooks v6.0.0) are documented examples, not a version allowlist or remote checks performed by the reviewer. SHA/tag resolution belongs to humans and CI. Other annotated-tag object SHAs can also be valid frozen refs; inability to check a remote is only a non-blocking informational note, never by itself a Critical finding. `repo: local` and `repo: meta` do not have remote revisions. Source trust and SHA pinning are not proof of safe code or execution isolation.

The system-hook exception covers a version-independent source/ID/language/entry contract: `openshift-hyperfleet/hyperfleet-hooks`, `language: system`, and `hyperfleet-gofmt` → `make gofmt`, `hyperfleet-golangci-lint` → `make lint`, or `hyperfleet-go-vet` → `make go-vet`. These combinations are documented in v0.3.0 but are not restricted to it. Review visible revision changes, consumer overrides/arguments, and changes to the delegated Make recipes and their dependencies. The exception does not approve new revisions automatically, allow arbitrary commands, or waive human review and CI. There is no corresponding blanket exception for local or script hooks.

This is a maintenance commitment for the contract, not every release: new sources or added/renamed system hooks, changed languages, or different entrypoints that need to extend the exception require a central policy PR before consumer rollout. Routine LeakTK or HyperFleet hooks version/SHA bumps preserving the existing source and hook contract do not require a policy PR or refreshed example SHAs. They remain subject to normal revision review and CI, but must not produce a Critical finding merely because the version is not listed here.

### Review regression cases

These are expected review outcomes, not an executable security scanner. YAML/schema validation can check configuration structure, but cannot prove that the reviewer follows natural-language instructions.

| Case | Expected outcome |
|------|------------------|
| LeakTK's documented Go hook, exact source, documented full SHA and frozen release comment | No generic untrusted-source finding; review revision changes normally |
| HyperFleet commitlint, exact source, documented frozen SHA, published Go entrypoint | No generic untrusted-source finding; no system-hook exception needed |
| Each of the three documented HyperFleet v0.3.0 system hooks with its exact published entrypoint and unchanged safe Make recipe | No finding solely for source trust or `language: system` |
| A newer LeakTK release with full SHA/frozen comment, same source and hook contract | Normal revision review; no policy update or finding solely for an unlisted version |
| A newer HyperFleet hooks release preserving the documented source/ID/system/entry combinations | Same exception applies; no central policy PR or Critical solely for a new version/SHA |
| The documented HyperFleet v0.3.0 frozen annotated-tag object SHA | Accept the documented ref; do not demand the peeled commit or claim a remote check |
| Another repo under any listed owner (including `pre-commit` or `astral-sh`), or a lookalike owner/host/repository | No owner-prefix exception; flag an untrusted origin |
| `rev: main`, a release tag, or an abbreviated SHA, even with a frozen comment or trusted source | Critical immutable-pinning finding |
| Full SHA with a missing or mismatched frozen comment | Update-metadata finding; not by itself evidence of malicious code |
| A new system hook ID, changed entrypoint, or consumer arguments invoking additional commands/targets | Outside the exception; flag unexpected execution |
| A local system hook reusing one of the three HyperFleet IDs | Not covered by the remote-source exception; review command execution |
| A documented target or its transitive recipe/script changed to execute unverified downloads, upload credentials, or interpolate untrusted shell input | Security finding remains required, regardless of hook source |
| Manifest or SHA/tag resolution unavailable remotely | At most a non-blocking informational note; not a Critical finding or a claim that the reviewer verified it |
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
