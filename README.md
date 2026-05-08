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
| Review profile | `assertive` — thorough feedback on every PR |
| Path instructions | Stricter review for `cmd/`, `config/`, `deploy/`, `charts/`, `migrations/` |
| Code guidelines | Reads `CLAUDE.md`, `AGENTS.md`, `.cursor/rules/*.mdc` from each repo |
| Linked repositories | Architecture, API, Sentinel, Adapter, Broker — cross-repo analysis |
| Tools | `golangci-lint`, `gitleaks`, `yamllint`, `markdownlint` enabled |
| JIRA | Enabled for `HYPERFLEET` project (requires Forge app — see TODO below) |
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

See the [CodeRabbit configuration docs](https://docs.coderabbit.ai/guides/configuration-overview) for all available options.

## Current limitations

### Learnable rules disabled

`knowledge_base.opt_out: true` is mandatory across all Red Hat CodeRabbit repos due to contractual data retention restrictions. This disables learnable rules (rules that improve from reviewer feedback over time). Under review by the PTLT team — no timeline for resolution.

### Web search disabled

`web_search.enabled: false` is set at the Red Hat enterprise level.

### JIRA comment-thread depth

CodeRabbit reads JIRA ticket descriptions and acceptance criteria but does **not** read comment threads on tickets. For deep JIRA validation (including refinements discussed in comments), use the Claude Code `/review-pr` skill.

## TODO: manual setup steps

### JIRA Forge app installation

The JIRA integration requires the CodeRabbit Forge app installed on our Jira instance. This is a one-time manual step:

1. Contact PTLT admins (Rory Thrasher or Adi Demback) to install the Forge app
2. An Installation Secret is generated in the CodeRabbit UI (expires in 15 minutes)
3. The Forge app is installed via Atlassian Marketplace using that secret
4. Once installed, CodeRabbit can link PRs to JIRA tickets and validate acceptance criteria

Until this is done, the `jira` configuration in `.coderabbit.yaml` is ready but inactive.

### CodeRabbit app permissions

Ensure the CodeRabbit GitHub app is installed on all linked repositories:

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
- [JIRA integration](https://docs.coderabbit.ai/integrations/jira)
- [Tool integrations](https://docs.coderabbit.ai/tools/list)
- [HyperFleet automated PR review strategy](https://github.com/openshift-hyperfleet/architecture/blob/main/hyperfleet/docs/automated-pr-review-strategy.md)
