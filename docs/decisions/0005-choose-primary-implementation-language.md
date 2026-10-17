---
status: proposed
date: 2026-10-17
decision-makers: [project maintainers]
consulted: []
informed: []
---

# Choose the primary implementation language

## Context and Problem Statement

The repository starts with specs, decisions and documentation, and no code.
Java was the leading candidate in the template this project started from. The [options research](../research/2026-10-17-tablet-home-dashboard-options.md) has since shown that the dashboard page runs in a browser on the wall tablet, so the page is TypeScript (or JavaScript) whatever is chosen here.
The open question is the language of anything that runs at home beside the page, which depends on the hub ([ADR-0007](0007-choose-the-integration-hub-for-home-devices-and-services.md)). With Home Assistant as the hub, the page may need no backend at all.
Choosing the language before knowing the workload (latency, integrations, deployment target, team skills) risks a decision that is costly to reverse.

This ADR stays `proposed` until it is decided. While it is open, contributors and agents must not add build files, source trees or language-specific tooling.

## Decision Drivers

- Fit for the first specified capabilities (to be filled in once `openspec/specs/` has content)
- Fit for a browser page on the wall tablet and, if needed, a small service at home
- Maintained libraries for the household's devices and services: the Home Assistant client, Sonos, Hue and the Google APIs
- Team skills and hiring
- Ecosystem maturity for the expected integrations
- Long-term maintainability and upgrade path (LTS releases)
- Quality of AI coding agent support (training coverage, tooling, fast feedback from compiler and tests)
- Build and test speed, so agents get feedback quickly

## Considered Options

- Java (current LTS) with Gradle or Maven
- Kotlin on the JVM
- TypeScript (the browser page, and Node.js for any backend)
- Go
- Python
- *(add options as they come up)*

## Decision Outcome

Not decided yet. The options research points to **TypeScript** for the page and for any backend beside it.
**Revisit when** the hub is chosen ([ADR-0007](0007-choose-the-integration-hub-for-home-devices-and-services.md)), and before the first OpenSpec change that needs code.

When deciding: set `status: accepted`, fill in the outcome and `decision-makers`, then in the same PR:

- add build files and source layout
- replace the placeholder commands in `AGENTS.md` ("Commands")
- add language entries to `.gitignore` and `.editorconfig`
- add a CI job for build and test
- update `docs/architecture/README.md` (constraints, solution strategy)
- update the language line in `AGENTS.md` ("Project overview"), `README.md` and `openspec/config.yaml`

### Consequences

- Good, because the structure, workflow and documentation can be used right away without committing to a stack.
- Bad, because agents have no build or test commands to run until this is decided.

## Pros and Cons of the Options

### Java (current LTS) with Gradle or Maven

- Good, because of the mature ecosystem, strong static typing and long LTS support windows.
- Good, because compiler and test feedback make agent-written code easier to verify.
- Neutral, because of the choice between Gradle and Maven, which needs its own ADR.
- Bad, because it is more verbose than the alternatives, which costs agent context and review effort.
- Bad, because it cannot run the page, so it would only be a thin backend next to a TypeScript page; no maintained Java Sonos library was found, and Home Assistant has no official Java client.

### Kotlin on the JVM

- Good, because it runs on the same ecosystem as Java with less boilerplate.
- Bad, because builds are slower and the hiring pool is smaller than Java's.

### TypeScript (the browser page, and Node.js for any backend)

- Good, because the documentation tooling already runs on Node.js, so the stack would be shared.
- Good, because the page is TypeScript anyway, so the page and any backend share one language, one type system and one test setup.
- Good, because Home Assistant's official client, `home-assistant-js-websocket`, is TypeScript, and the owner already builds web apps with TypeScript, React and Vite.
- Bad, because type safety is weaker at runtime.

### Go

- Good, because of simple deployment and fast builds.
- Bad, because the domain-modelling features are more limited.

### Python

- Good, because Home Assistant and most device libraries are written in Python, so it is the language for a custom Home Assistant integration if one is ever needed.
- Bad, because it does not run in the browser, so the page would still be TypeScript.

## More Information

Open question tracked here instead of in an issue, so that agents see it in the decision log.

- 2026-10-17: updated with the findings of the [options research](../research/2026-10-17-tablet-home-dashboard-options.md); the question is still open.
