# Contributing to Vexum Core

Thanks for your interest in contributing. Vexum Core is an open-source, Apache-2.0,
self-hostable meeting-intelligence runtime, contributed to the Fintech Open Source
Foundation (FINOS).

## Developer Certificate of Origin (DCO)

All commits must be signed off under the
[Developer Certificate of Origin](https://developercertificate.org/). Add a
`Signed-off-by` line to each commit:

    git commit -s -m "your message"

Pull requests whose commits are not signed off cannot be merged.

## License

This project is licensed under **Apache-2.0**. By contributing, you agree your
contributions are licensed under the same terms (inbound = outbound). Any
third-party code you introduce must be Apache-2.0-compatible.

## How to contribute

- All changes land via **pull request**, reviewed by a maintainer (see
  [MAINTAINERS.md](MAINTAINERS.md)).
- For substantial changes, open an issue or discussion first so the approach can
  be agreed before implementation.
- Keep PRs focused — one logical change per PR.

## Repository structure

Vexum Core is microservices, each internally a modular monolith, contract-bounded
at two scales (published JSON-Schema contracts between services, ports within).
Work in the domain your change belongs to:

| Path | Domain |
|---|---|
| `core/runtime/` | kernel — spawn/execute workloads + mount the workspace |
| `core/meetings/` | capture — join → capture → transcript |
| `core/agent/` | execution — transcript → governed action |
| `core/identity/` | accounts · auth · tokens · audit |
| `core/gateway/` | the edge — auth · routing · WS fan-out |
| `clients/` | terminal · slim |
| `deploy/` · `docs/` | deployment topologies · documentation |

Published contracts live with their owner domain (`core/<domain>/contracts/`).
Cross-domain boundaries are the versioned contracts and must not be bypassed.

## Scope

This repository is the **neutral, self-hostable runtime**. Hosted operations,
billing, and any branded application layer live outside this project and consume
it via the published contracts — please keep that boundary clean in PRs.

## Building and testing

See the [documentation](https://docs.core.vexa.ai) and the repository `Makefile`
and `deploy/` topologies. An artifact "exists" only when the project's automated
gates are green.

## Governance

This project follows FINOS governance. See the
[FINOS Contribution Process](https://community.finos.org/docs/governance/Software-Projects/contribution/)
and [Maintainer Responsibilities](https://community.finos.org/docs/finos-maintainers-cheatsheet/).
