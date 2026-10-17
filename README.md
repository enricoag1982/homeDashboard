# Home Dashboard

A web page shown full screen on a wall-mounted tablet at home. It shows and controls the household's lights, Sonos speakers, Alexa (Echo) devices and Nest thermostat, and shows the family calendar, shared lists and the weather forecast.

> [!NOTE]
> **Status:** exploring options, nothing is built yet. The [options research](docs/research/2026-10-17-tablet-home-dashboard-options.md) compares the routes for each device and service, the hub, the tablet and the language. The main choices are open decisions in the [decision log](docs/decisions/README.md); the rest are next steps in the research.

## Where things stand

| Question | Leaning from the research | Decision |
| --- | --- | --- |
| Is a web page alone enough? | No: an always-on hub at home is needed to reach the devices and keep secrets off the tablet | [ADR-0007](docs/decisions/0007-choose-the-integration-hub-for-home-devices-and-services.md), proposed |
| Which hub? | Home Assistant, with a custom dashboard page on top | [ADR-0007](docs/decisions/0007-choose-the-integration-hub-for-home-devices-and-services.md), proposed |
| Which tablet? | Samsung Galaxy Tab with Fully Kiosk Browser | [ADR-0008](docs/decisions/0008-choose-the-wall-tablet-and-kiosk-browser.md), proposed |
| Which language? | TypeScript | [ADR-0005](docs/decisions/0005-choose-primary-implementation-language.md), proposed |

## How this repository works

It started from [openSpecProjectStructure](https://github.com/enricoag1982/openSpecProjectStructure): behavior is specified with [OpenSpec](https://github.com/Fission-AI/OpenSpec), decisions are [MADR](https://adr.github.io/madr/) records, and the docs are plain Markdown that is also an [Obsidian](https://obsidian.md/) vault.
Start with [AGENTS.md](AGENTS.md), whether you are a person or an AI agent.

Requirements: Node.js 20.19 or later, and git.

```bash
npm ci          # install doc and spec tooling
npm run check   # validate specs, lint Markdown, check decisions, links and review dates
```

The full loop is in the [development workflow](docs/process/development-workflow.md).

> [!WARNING]
> This repository is public. Never commit tokens, API keys, calendar or list contents, addresses, network details or other personal data.

## Documentation map

- [Options research](docs/research/2026-10-17-tablet-home-dashboard-options.md) · [Decision log](docs/decisions/README.md) · [Principles](docs/principles.md) · [Glossary](docs/glossary.md) · [Architecture](docs/architecture/README.md)
- Process: [Development workflow](docs/process/development-workflow.md) · [Documentation lifecycle](docs/process/documentation-lifecycle.md) · [Working with AI agents](docs/process/working-with-agents.md)
