---
status: proposed
date: 2026-10-17
decision-makers: []
consulted: []
informed: []
---

# Choose the wall tablet and kiosk browser

## Context and Problem Statement

The dashboard runs full screen, all day, on a tablet mounted on the wall. The tablet has not been bought yet.
The choice decides which browser engine the page must support, how the screen wakes and sleeps, how the tablet is locked to the page, and how the battery survives being charged permanently.
The options and their sources are in the [options research](../research/2026-10-17-tablet-home-dashboard-options.md).

## Decision Drivers

- Battery safety when charging all the time (a charge limit the tablet enforces)
- Kiosk control: locked to the page, survives restarts, screen on and off by schedule or motion, remote reload
- A current, updatable browser engine
- Price and availability in the United States
- Whether the page can be tested with the same engine in CI

## Considered Options

- Samsung Galaxy Tab (A11+ or S10 FE) with Fully Kiosk Browser
- iPad (A16) with a Home Screen web app and Guided Access or Single App Mode
- Amazon Alexa Tablet (8, 11 or 12 Pro), the Android-based successor to Fire tablets
- An in-wall Android panel (Sonoff NSPanel Pro class)

## Decision Outcome

Not decided yet. The research recommends a **Samsung Galaxy Tab with Fully Kiosk Browser PLUS**, with the 80% charge limit on, an external motion sensor for screen on and off, and power from an in-wall PoE-to-USB-C adapter.
**Revisit when** the hub is chosen ([ADR-0007](0007-choose-the-integration-hub-for-home-devices-and-services.md)) and before buying hardware.

When deciding, set `status: accepted`, fill in the outcome and `decision-makers`, and in the same pull request update `docs/architecture/README.md` (constraints, deployment view), the tablet lines in `AGENTS.md`, `README.md` and `openspec/config.yaml`, and the Wall tablet entry in `docs/glossary.md`.

### Consequences

- Good, because the page can be built against web standards first and tested on the chosen tablet later.
- Bad, because browser-specific behavior (wake lock, autoplay, certificates) cannot be specified until the tablet is chosen.

## Pros and Cons of the Options

### Samsung Galaxy Tab with Fully Kiosk Browser

- Good, because One UI 6.1 and later can stop charging at 80%, which protects the battery.
- Good, because Fully Kiosk Browser PLUS (€8.90 per device, one time) locks the tablet to the page, can turn the screen on and off from Home Assistant (the research prefers an external motion sensor to its camera motion detection), offers a REST API and has a Home Assistant integration.
- Good, because the Android System WebView updates through Google Play, and Chromium can run the same engine in CI.
- Bad, because Fully Kiosk's remote-admin port is reachable on the home network and must be protected with a password.

### iPad with a Home Screen web app

- Good, because Safari supports the Screen Wake Lock API (also in Home Screen web apps since iOS 18.4), and iPadOS 26 opens Home Screen sites as web apps by default.
- Good, because newer iPads can limit charging to 80%.
- Bad, because Guided Access ends when the iPad restarts; Single App Mode survives restarts but needs Apple Configurator or a device-management service.
- Bad, because Safari only trusts a self-signed certificate on the home network after it is installed as a profile, and there is no remote control of the screen comparable to Fully Kiosk.

### Amazon Alexa Tablet

- Good, because it runs Android 16 with Google Play, so Fully Kiosk Browser can run on it.
- Bad, because it launched on 8 October 2026 (from $229.99), and nothing was found yet about a charge limit or a hub mode.
- Bad, because older Fire OS tablets had a WebView that could not be updated, and Fire tablet production has ended.

### An in-wall Android panel

- Good, because it mounts flush in the wall and needs no separate power adapter.
- Bad, because the NSPanel Pro class runs Android 8.1, needs a WebView update over ADB and is slow, which suits only a secondary controller.

## More Information

- Research: [tablet home dashboard options](../research/2026-10-17-tablet-home-dashboard-options.md)
- Prices were checked in October 2026 and changed several times during the year; check them again before buying.
