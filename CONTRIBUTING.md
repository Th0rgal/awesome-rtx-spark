# Contribution Guidelines

Please note that this project is released with a [Contributor Code of Conduct](CODE_OF_CONDUCT.md). By participating in this project you agree to abide by its terms.

## Adding a resource

- Open a pull request that edits `README.md`, adding one entry per pull request where possible.
- Use the format `- [Name](link) - Description.` with one short, original sentence ending in a period.
- Link the original source: the repository, patch, maintainer report or measurement — not a repost or summary.
- Say which platform the resource covers: native N1X Linux, Windows on Arm, WSL2, or GB10 desktop (DGX Spark). Shared architecture alone does not show that something works on an N1X laptop.
- Mark experimental or unverified support with *(experimental)* or *(unverified)*.
- Keep Linux kernels and CUDA compute kernels in clearly separate places.
- Prefer one excellent entry over several similar ones. Archived or superseded projects should point to their successor.

## Reporting hardware or benchmark results

Use the [hardware or benchmark report](https://github.com/th0rgal/awesome-rtx-spark/issues/new?template=hardware-or-benchmark-report.yml) issue form. Compatibility details go in [docs/linux.md](docs/linux.md) and measurements in [docs/benchmarks.md](docs/benchmarks.md), always with an evidence link. Proposed experiments are welcome but are listed separately from completed results.

## Checks

CI runs [awesome-lint](https://github.com/sindresorhus/awesome-lint) and the [lychee](https://github.com/lycheeverse/lychee) link checker. To run them locally:

```sh
npx awesome-lint README.md
lychee --config lychee.toml README.md docs/*.md
```
