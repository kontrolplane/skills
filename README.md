# kontrolplane/skills

A collection of skills for cloud, devops and platform engineering tooling. Skills are markdown files that provide AI coding assistants with domain-specific knowledge. They help with gotchas, patterns, and troubleshooting that aren't always obvious from documentation alone.

## included skills

| Skill | Description |
|-------|-------------|
| **argocd** | GitOps continuous delivery—sync policies, ApplicationSets, troubleshooting OutOfSync states |
| **grafana** | Dashboard JSON configuration, panel setup, template variables, alerting rules |
| **kubernetes** | Pod troubleshooting, resource limits, probes, RBAC, NetworkPolicies |
| **kyverno** | Policy validation/mutation/generation, pattern matching, JMESPath expressions |
| **loki** | LogQL queries, Promtail pipelines, metric queries from logs |
| **prometheus** | PromQL queries, rate/histogram functions, alerting and recording rules |
| **terraform** | HCL patterns, state management, count vs for_each, module composition |

## installation

```bash
npx skills kontrolplane/skills
```

## skill philosophy

These skills focus on:

- **Gotchas and traps** — Non-obvious behavior that causes real problems
- **Troubleshooting patterns** — Diagnostic workflows with relevant commands
- **Configuration snippets** — Copy-paste-ready examples for common patterns
- **Conceptual clarity** — Explaining distinctions like `requests` vs `limits` or `rate()` vs `irate()`

They intentionally avoid:

- Comprehensive CLI references (use `--help` or official docs)
- Tutorial-style introductions (assumes working knowledge)
- Exhaustive API documentation

## contributing

1. Create a directory: `skills/<skill-name>/`
2. Add `SKILL.md` with frontmatter and content
3. Focus on gotchas, patterns, and troubleshooting

### content guidelines

- **Focus on gotchas** — What trips people up? What's non-obvious?
- **Include targeted commands** — CLI commands belong when they're part of a debugging workflow
- **Keep it scannable** — Tables, code blocks, and short sections work better than prose
- **Keep `SKILL.md` concise** — Move detailed reference to supporting files if needed

### supporting files

Skills can include additional files for templates, examples, or scripts:

```
<skill-name>/
├── SKILL.md           # Main instructions (required)
├── reference.md       # Detailed docs (loaded when needed)
├── examples/          # Example configurations
└── scripts/           # Utility scripts
```
