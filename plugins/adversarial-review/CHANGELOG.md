# Changelog

All notable changes to the `adversarial-review` plugin are documented here.
This project follows [Semantic Versioning](https://semver.org): bump the
`version` in both `.claude-plugin/marketplace.json` and the plugin's
`plugin.json` together when you publish.

## [1.0.0] — 2026-06-18

### Added

- Initial release.
- 5-pass adversarial audit: claim extraction, source tracing, source
  interrogation, self-source (JIRA) trap detection, and silence/omission pass.
- Verdict system: `VERIFIED` / `UNVERIFIED` / `UNRELIABLE — SELF-SOURCED` /
  `CONTRADICTED` / `FABRICATION RISK`.
- Numbered findings (`F-N`) for precise feedback, assumptions inventory,
  alternative explanations with distinguishing tests, and an investigation log.
- `references/patterns.md` with worked examples of each failure pattern.
