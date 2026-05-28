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
