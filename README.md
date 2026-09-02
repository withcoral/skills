# Coral Skills

<!-- AUTO-GENERATED from withcoral/coral plugins/coral/skills. Do not edit directly. -->

Agent skills for [Coral](https://withcoral.com) - one SQL interface over APIs, files, and live sources, built for agents.

## Installation

### Claude Code

```
/plugin marketplace add withcoral/skills
/plugin install coral@withcoral
/plugin install coral-sources@withcoral
```

`coral` is the Coral query skill. `coral-sources` adds the source-spec authoring
and review skills - install it only if you write or review Coral sources.

### Other agents

```bash
npx skills add withcoral/skills
```

## Available Skills

| Skill | Description |
|-------|-------------|
| [`coral`](coral/SKILL.md) | Query live sources through Coral MCP. Use when the task needs GitHub, Jira, Slack, Linear, Datadog, Sentry, files, or connected data. |
| [`coral-create-source-spec`](coral-create-source-spec/SKILL.md) | Create or update a Coral source spec YAML for a custom HTTP API or local dataset. Use when authoring a standalone source for `coral source add --file`, or when adapting that spec into a Coral repo source under `sources/core` or `sources/community`. |
| [`coral-review-source-spec`](coral-review-source-spec/SKILL.md) | Review new or updated Coral source manifests and source PRs for content, style, product fit, query ergonomics, documentation quality, and consistency with existing Coral sources. Use when Codex is asked to review a sources/core/name or sources/community/name source directory, a manifest.yaml, or a GitHub PR that adds or changes a Coral source. |

## License

Apache 2.0 - see [LICENSE](LICENSE).
