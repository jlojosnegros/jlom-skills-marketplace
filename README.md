# jlom-skills

A personal [Claude Code](https://code.claude.com) plugin marketplace. Add it
once on any machine and install the plugins below; update them all with a single
command when this repo changes.

## Install

In Claude Code, add this repo as a marketplace (replace with your GitHub
`owner/repo`):

```text
/plugin marketplace add jlojosnegros/jlom-skills-marketplace
```

Then install a plugin from it:

```text
/plugin install adversarial-review@jlom-skills
```

> `@jlom-skills` is the marketplace name — it comes from the `name` field in
> `.claude-plugin/marketplace.json`, not from the repo name. If you rename the
> marketplace, update the install command to match.

To pull the latest version after this repo changes:

```text
/plugin marketplace update jlom-skills
```

You can also do this non-interactively from a shell, which is handy for
provisioning new machines or CI:

```bash
claude plugin marketplace add jlojosnegros/jlom-skills-marketplace
claude plugin install adversarial-review@jlom-skills
```

## Plugins

| Plugin                                             | What it does                                                                                                                                                                                                                                                                               |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`adversarial-review`](plugins/adversarial-review) | Adversarially audits a finished report, design doc, PR/MR review, or JIRA analysis **before you act on it** — catching fabricated facts, unverified numbers, partially-read or unreliable sources, and self-referential claims (the agent treating its own earlier JIRA comment as truth). |
| [`agentdoc`](plugins/agentdoc)                     | Generates and maintains AI-agent-oriented documentation for any repository — a lightweight `CLAUDE.md` plus a deep `agent-overlay.md`, with drift detection, freshness scoring, and protection for human-written sections. Ships a CI health-check script.                                 |

## Repository layout

```text
.
├── .claude-plugin/
│   └── marketplace.json        # Catalog Claude Code reads to discover plugins
├── plugins/
│   ├── adversarial-review/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── skills/adversarial-review/
│   │   │   ├── SKILL.md
│   │   │   └── references/patterns.md
│   │   ├── README.md
│   │   └── CHANGELOG.md
│   └── agentdoc/
│       ├── .claude-plugin/plugin.json
│       ├── skills/agentdoc/
│       │   ├── SKILL.md
│       │   └── scripts/agentdoc-check.sh
│       └── README.md
├── LICENSE
└── README.md
```

The marketplace catalog and each plugin are independent: this repo can grow to
host more plugins by adding folders under `plugins/` and entries to
`marketplace.json`.

## Adding another plugin later

1. Create `plugins/<new-plugin>/.claude-plugin/plugin.json` and a
   `skills/<skill-name>/SKILL.md`.
2. Add an entry to the `plugins` array in `.claude-plugin/marketplace.json`
   pointing at `./plugins/<new-plugin>`.
3. Commit and push. Installed clients pick it up via
   `/plugin marketplace update jlom-skills`.

## License

See [LICENSE](LICENSE).
