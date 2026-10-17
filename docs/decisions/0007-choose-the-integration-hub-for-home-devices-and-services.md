---
status: proposed
date: 2026-10-17
decision-makers: []
consulted: []
informed: []
---

# Choose the integration hub for home devices and services

## Context and Problem Statement

The dashboard must show and control Philips Hue and Wi-Fi lights, Sonos speakers, Alexa (Echo) devices and a Google Nest thermostat, and show Google Calendar, shared lists and the weather forecast.
A web page on its own cannot do most of this: browsers block an HTTPS page from calling HTTP devices on the home network, cannot discover devices (no mDNS or SSDP), and should not hold OAuth refresh tokens or API keys on the wall tablet. Amazon offers no official API to control Echo devices, and Google Keep has none for personal accounts.
The research therefore assumes that something always on must run at home and integrate the devices and services; the first option below tests that assumption. The household has no such computer yet.
Which hub should it be? The options and their sources are in the [options research](../research/2026-10-17-tablet-home-dashboard-options.md).

## Decision Drivers

- Coverage of the household's devices and services, with official APIs where they exist
- Effort to build and keep working: every integration someone else maintains is one the project does not
- Freedom to design the dashboard page, which is the point of the project
- Keeping secrets off the wall tablet and working on the home network when the internet is down
- Hardware and running cost

## Considered Options

- A web page alone, with no hub at home
- Home Assistant as the hub, with Home Assistant dashboards
- Home Assistant as the hub, with a custom dashboard that uses its WebSocket API
- A custom backend that integrates each service directly, without Home Assistant
- An off-the-shelf product (Echo Hub or Echo Show, Homey Pro, Hubitat, DAKboard)

## Decision Outcome

Not decided yet. The research recommends **Home Assistant as the hub, with a custom dashboard**, built on a working Home Assistant install so that Home Assistant dashboards remain a fallback.
**Revisit when** the owner has reviewed the research, and before the first OpenSpec change that needs code.

When deciding, set `status: accepted`, fill in the outcome and `decision-makers`, and in the same pull request update [ADR-0005](0005-choose-primary-implementation-language.md), `docs/architecture/README.md` (constraints, solution strategy, deployment view), the hub lines in `AGENTS.md`, `README.md` and `openspec/config.yaml`, and the Hub entry in `docs/glossary.md`.

### Consequences

- Good, because the project can start with specs and a hardware purchase plan without committing to a hub.
- Bad, because no capability that touches devices can be specified in detail until the hub is chosen.

## Pros and Cons of the Options

### A web page alone, with no hub at home

The page is hosted anywhere (for example GitHub Pages) and calls each service from the browser.

- Good, because there is no hardware to buy or keep running.
- Good, because weather (National Weather Service, Open-Meteo) works this way.
- Bad, because the browser blocks or cannot reach most devices: mixed content and Local Network Access rules, no LAN discovery, no CORS on most device APIs.
- Bad, because Google's browser-only sign-in issues no refresh token, so an unattended tablet loses Google access, and Sonos and Nest events need a server to receive them.
- Bad, because every API key and token would sit on the wall tablet.

### Home Assistant as the hub, with Home Assistant dashboards

Home Assistant on a Home Assistant Green (about $199 in October 2026), a Raspberry Pi 5 or a mini PC, with Sections dashboards and community cards (Mushroom, Bubble Card) in kiosk mode.

- Good, because built-in integrations cover nearly the whole list: Hue and Sonos locally with push updates, Nest through Google's Smart Device Management API, Google Calendar, Google Tasks, National Weather Service, and Alexa through the built-in, unofficial Alexa Devices integration.
- Good, because a working dashboard is possible in days, and Home Assistant ships monthly releases.
- Bad, because the page's design is limited to what cards and themes allow, which leaves little to build.
- Bad, because Wi-Fi lights that exist only in the Alexa app are not reachable through either Alexa integration; they need their vendor's integration, Matter sharing, or Alexa routines. This applies to every option.

### Home Assistant as the hub, with a custom dashboard

The same hub; the dashboard is a custom web app served by Home Assistant (or next to it) that uses the official `home-assistant-js-websocket` client to subscribe to entities and call services.

- Good, because the page talks to one API, and Home Assistant keeps the device and cloud integrations working.
- Good, because the page can be designed freely, and Home Assistant's own login flow gives short-lived, revocable tokens.
- Neutral, because the WebSocket API has no formal stability promise, although Home Assistant's own frontend depends on it.
- Neutral, because the research did not establish whether Home Assistant can limit the wall tablet's user to specific entities; it may be able to control everything.
- Bad, because the project must keep two things running: the Home Assistant install and the custom page.

### A custom backend that integrates each service directly

- Good, because the project owns every line and has no dependency on Home Assistant releases.
- Bad, because it re-implements OAuth flows, device discovery, push events and error handling for every service, which is by far the most work.
- Bad, because the device libraries are strongest in Python, adequate in Node.js and weakest in Java (no maintained Java Sonos library was found).

### An off-the-shelf product

- Good, because there is nothing to build.
- Bad, because no product found covers the whole list with real design control: the Echo Hub and Echo Show screens are built from Amazon's own widgets, Homey Pro costs $449, DAKboard and MagicMirror² are displays rather than home-control panels, and none of them reads Google Keep.

## More Information

- Research: [tablet home dashboard options](../research/2026-10-17-tablet-home-dashboard-options.md)
- Related: [ADR-0005](0005-choose-primary-implementation-language.md) (implementation language) and [ADR-0008](0008-choose-the-wall-tablet-and-kiosk-browser.md) (wall tablet and kiosk browser)
