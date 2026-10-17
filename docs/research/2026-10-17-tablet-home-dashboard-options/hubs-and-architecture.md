---
status: concluded
date: 2026-10-17
authors: [Claude Code]
related: [2026-10-17-tablet-home-dashboard-options]
---

# Hubs and architecture for a self-built wall-tablet home dashboard (October 2026)

Research date: 2026-10-17. Scope: the hub/architecture layer only (device APIs, Google data, weather and tablet kiosk setup are covered by other researchers).

Method note for the report writer: WebFetch could not resolve several primary domains in this session (home-assistant.io, developers.home-assistant.io, nabucasa.com, developers.google.com, developer.chrome.com, raspberry.tips). Where that happened I used (a) the raw Markdown/JSON source of the same official docs on GitHub (the home-assistant.io and developers.home-assistant.io sites are built from those repos, and integration `manifest.json` files are the canonical source of IoT class and library), and (b) search-engine summaries of the primary page, marked "(search summary)". GitHub star counts and last-push dates come from the GitHub search API on 2026-10-17.

## Q1. Why a pure in-browser page (no backend) mostly does not work

### Takeaway

A static page on the tablet cannot reliably talk to the owner's devices and cloud accounts by itself: browsers block HTTPS-to-HTTP LAN calls (mixed content), Chrome now gates public-site-to-LAN requests behind a permission prompt (Local Network Access, Chrome 142, late 2025), web pages have no UDP so they cannot do mDNS/SSDP discovery, most device and cloud APIs do not send CORS headers for arbitrary origins, and Google's browser-only OAuth model issues no refresh token, so an unattended wall tablet would lose Google access within about an hour. Some process other than the page has to hold the long-lived credentials and talk to the devices; that is either a hub (Home Assistant and similar) or the owner's own backend.

### Cited Findings

- Browsers split mixed content into "upgradable" (images, audio, video, which are auto-upgraded to HTTPS) and "blockable"; `fetch`/XHR from an HTTPS page to an `http://` URL is in the blocked group — [MDN Mixed content](https://developer.mozilla.org/Security/MixedContent) (search summary).
- Only loopback (`localhost`, `127.0.0.1`) is treated as "potentially trustworthy" and exempt; a private LAN IP over plain HTTP is still mixed content. Firefox added the localhost exception in Firefox 55 — [Mozilla bug 903966](https://bugzil.la/903966); browser behaviour for 127.0.0.1 historically varied — [WebKit bug 171934](https://www2.webkit.org/show_bug.cgi?id=171934).
- **Changed in 2025:** Chrome 142 (stable late 2025) added a Local Network Access (LNA) permission prompt: a public website must get explicit user permission before making requests to devices on the local network; if denied, the fetch fails with a network error. The permission is only requested from HTTPS (secure) pages; requests from a local origin to a local target ("same address space") do not prompt — [OpenReplay: Chrome Local Network Access permission](https://blog.openreplay.com/chrome-local-network-access-lna-permission/); [University of Illinois IT KB](https://help.uillinois.edu/TDClient/37/uic/KB/ArticleDet?ID=2993); [Beyond Identity: Managing the Chrome v142 LNA prompt](https://support.beyondidentity.com/hc/en-us/articles/34457691192215-Managing-the-Chrome-v142Local-Network-Access-Prompt). (Secondary sources; Google's own Chrome blog page could not be fetched.)
- Web pages have no UDP interface, so SSDP (UPnP, used by Sonos) and mDNS discovery cannot be done from page JavaScript; Chrome's `chrome.udp.sockets` is extension/app-only — [j2inet: SSDP Discovery in HTML (2018)](https://j2inet.blog/2018/12/28/ssdp-discovery-in-html/). A W3C "Networked Service Discovery" browser API was drafted in 2012 and reviewers raised the security problem of letting JavaScript run mDNS/UPnP queries — [W3C public-device-apis, Aug 2012](https://lists.w3.org/Archives/Public/public-device-apis/2012Aug/0009.html). I found no evidence it ever shipped.
- Google Identity Services "token model" (pure browser): access token issued, **no refresh token**; "a user gesture such as button press or clicking on a link is required to request and obtain a new, valid access token." The "code model" returns an authorization code that a backend exchanges for tokens, including a refresh token "which can be used to obtain access tokens without the user being present" — [Google: Use Code Model](https://developers.google.com/identity/oauth2/web/guides/use-code-model) and [Choose an authorization model](https://developers.google.cn/identity/oauth2/web/guides/choose-authorization-model?hl=zh-tw) (search summary).
- Home Assistant's own HTTP server does support CORS for named origins (`cors_allowed_origins`: "Origins that may make cross-origin requests"), TLS certificates, reverse-proxy trust and IP banning — [HA http integration docs (source)](https://raw.githubusercontent.com/home-assistant/home-assistant.io/current/source/_integrations/http.markdown). So a page served from a different origin can call HA, which is not true of most consumer devices.
- Community authors of custom HA dashboards flag that putting a long-lived HA token in browser code is a security risk for production use — [search summary of community.home-assistant.io "React dashboard" thread](https://community.home-assistant.io/t/react-dashboard/208107) and HA Component Kit docs ([GitHub](https://github.com/shannonhochkins/ha-component-kit)).

### Inferences

- The practical pattern is: the page talks to **one** origin that the owner controls (Home Assistant, or the owner's backend) and that origin holds the device connections and the OAuth refresh tokens. If that origin also serves the page (same origin), CORS, mixed content and the LNA prompt all disappear. Example: serve the page from HA itself (add-on, `www` folder) or from the owner's backend on the LAN.
- Serving the page over plain HTTP from a LAN server avoids mixed content but makes the page a non-secure context; some browser APIs (service workers, PKCE via `crypto.subtle`, the LNA prompt itself) require a secure context. A local TLS certificate (or HA served through HTTPS) is the cleaner long-term answer. (home-assistant-js-websocket only uses S256 PKCE "in a secure context", see Q3.)
- Storing secrets on the tablet: with a hub, the tablet only needs one credential (an HA refresh token or a long-lived token for a dedicated HA user), not Google/Amazon/Sonos credentials. That limits the blast radius if the tablet is stolen.

### Gaps

- Could not fetch Google's Chrome LNA article or the Chrome Status entry directly; whether Chrome on Android (the likely tablet browser) and Android WebView-based kiosk browsers (Fully Kiosk) enforce LNA identically is unverified. The kiosk researcher should check.
- No current (2025-2026) primary source found on CORS headers for specific devices (Hue bridge, Sonos); left to the device-API researcher.

## Q2. Option A: Home Assistant as hub plus its own dashboards (Lovelace)

### Takeaway

Home Assistant (HA) is the most complete free hub for the owner's device list, and its dashboards have improved a lot in 2024-2026 (Sections grid view is the default; a new auto-generated "Home" dashboard became the default for new installs in 2026.2). With HACS custom cards (Mushroom, Bubble Card) and kiosk helpers (kiosk-mode, wallpanel, browser_mod) it produces a good wall panel with no code, but customization is through YAML/card configuration and CSS, not a design the owner fully controls. Hardware costs rose sharply in 2026 because of the DRAM shortage: Home Assistant Green is now $199 (was $99), Raspberry Pi 5 8GB is about $175.

### Cited Findings

#### Release cadence and current version

- "New versions of Home Assistant are released on the first Wednesday of every month"; the last week of each cycle is focused on beta testing — [HA FAQ: release](https://home-assistant.io/faq/release) (search summary).
- Current release is **2026.10**, out since 7 October 2026 (rebuilt map card on OpenStreetMap vector tiles via a new "Map tiles" integration; trigger-ID UI) — [HA full changelog 2026.10](https://www.home-assistant.io/changelogs/core-2026.10/); [smarthomeassistent.de summary](https://www.smarthomeassistent.de/home-assistant-2026-10-das-sind-die-neuerungen/). Community advice: read the backward-incompatible changes section and take an external backup before upgrading — [simon42 community thread](https://community.simon42.com/t/probleme-mit-home-assistant-update-ab-version-2026-10-aktuell-2026-10-0-video-von-simon42/93738).
- HA core: ~91.6k GitHub stars, Python, pushed 2026-10-17 — [home-assistant/core](https://github.com/home-assistant/core).

#### Dashboards (changed 2024-2026)

- View types: Sections is the **default** view ("Arranges cards in a grid system and lets you group them in sections"); others are Masonry, Sidebar, Panel — [HA docs: Dashboard views](https://www.home-assistant.io/dashboards/views/) (search summary). Sections arrived in 2024.3 and is now the default on new installs, with drag-and-drop grid placement — [housecontrollers.de, July 2026](https://www.housecontrollers.de/smart-home/home-assistant-dashboards/) (search summary). Some users find Sections wastes space and keep Masonry plus a layout-card — [simon42 community](https://community.simon42.com/t/luxus-home-assistant-dashboard-wie-auf-dem-bild-umsetzbar/89304).
- **2026.1:** the "Home" dashboard shows summary cards (lights, climate, security, media players, weather, energy) at the top on mobile; new Devices page for unassigned devices — [HA blog 2026.1](https://www.home-assistant.io/blog/2026/01/07/release-20261/); [AlternativeTo](https://alternativeto.net/news/2026/1/home-assistant-2026-1-brings-home-dashboard-improvements-and-more-purpose-specific-triggers).
- **2026.2:** the new Home dashboard is the official default for new installations; users who never changed their default are offered the switch; the old layout remains as a legacy template — [HA blog 2026.2 "Home, sweet overview"](https://www.home-assistant.io/blog/2026/02/04/release-20262/) (search summary); [PCWorld](https://www.pcworld.com/article/3053761/home-assistants-default-dashboard-got-a-major-glow-up.html).
- 2026.6 reportedly added card suggestions based on selected devices/entities and a card picker with live preview — [mobiflip.de](https://www.mobiflip.de/home-assistant-krempelt-seine-dashboards-komplett-um/amp/) (secondary; not confirmed in official notes).

**Popular custom cards and kiosk helpers (installed through HACS)** — GitHub stats as of 2026-10-17:

- HACS (Home Assistant Community Store): 7.8k stars, pushed 2026-10-16 — [hacs/integration](https://github.com/hacs/integration).
- Mushroom cards (TypeScript): 5.2k stars, pushed 2026-10-05; "popular collection for compact, touch-friendly interfaces" — [piitaya/lovelace-mushroom](https://github.com/piitaya/lovelace-mushroom); [luisllamas.es](https://www.luisllamas.es/en/creating-dashboards-home-assistant-lovelace/).
- Bubble Card (JavaScript): 4.7k stars, pushed 2026-10-14; mobile-first design with bottom nav bar and pop-ups — [Clooos/Bubble-Card](https://github.com/Clooos/Bubble-Card); [joinhomeshift examples](https://joinhomeshift.com/home-assistant-dashboard-examples/).
- kiosk-mode (hides header/sidebar): 817 stars, pushed 2026-10-16 — [NemesisRE/kiosk-mode](https://github.com/NemesisRE/kiosk-mode).
- Wallpanel (wall panel mode + photo screensaver): 883 stars, pushed 2026-09-29 — [j-a-n/lovelace-wallpanel](https://github.com/j-a-n/lovelace-wallpanel).
- browser_mod ("turn your browser into a controllable" device: popups, navigation, per-browser targeting): 1.8k stars, pushed 2026-10-07 — [thomasloven/hass-browser_mod](https://github.com/thomasloven/hass-browser_mod).
- Dwains Dashboard (auto-generated tablet/desktop UI): 2.1k stars, pushed 2026-09-04 — [dwainscheeren/dwains-lovelace-dashboard](https://github.com/dwainscheeren/dwains-lovelace-dashboard).
- matt8707/hass-config (a famous custom tablet design): 5.2k stars but last push 2024-08-13, i.e. stale — [matt8707/hass-config](https://github.com/matt8707/hass-config).

#### Install options and hardware costs (US, 2026; all affected by the DRAM shortage)

- **Home Assistant Green**: launched at $99 (2023), raised to $159, now $199 (double the original); Nabu Casa's Jan 2026 post says RAM prices drove it and "the component price of building a Home Assistant Green has nearly doubled" — [Nabu Casa: Green pricing change (2026-01-08)](https://www.nabucasa.com/news/2026-01-08-green-pricing-change/); [Notebookcheck](https://www.notebookcheck.net/Rising-costs-in-the-Smart-Home-The-impact-of-the-chip-and-RAM-shortage.1338441.0.html); [Matter Alpha](https://www.matteralpha.com/news/home-assistant-green-price-hiked-here-s-why) ($199 / EUR 179). Exact effective date of the $199 MSRP not confirmed; one site gives $199 as of July 2026 — [raspberry.tips](https://raspberry.tips/en/raspberrypi-tutorials/home-assistant-hardware-comparison-2026-green-blue-yellow-vs-raspberry-pi) (search summary). Some 2026 articles still quote $99 (outdated) — [Reolink blog](https://reolink.com/blog/home-assistant-green-vs-yellow/).
- **Home Assistant Yellow**: discontinued (one source: out of production October 2025), software support continues "as long as it's possible"; Nabu Casa points new buyers to Green; no successor announced as of July 2026 — [Matter Alpha explainer](https://www.matteralpha.com/explainer/home-assistant-yellow-is-discontinued-here-are-the-alternatives); [raspberry.tips](https://raspberry.tips/en/raspberrypi-tutorials/home-assistant-green-vs-yellow-comparison).
- **Raspberry Pi 5** (board only; needs PSU, case, SSD/SD): 8GB launched at $80 -> $95 (Dec 2025) -> $125 (Feb 2026) -> about $175 (April 2026); 16GB $120 -> $205 -> about $305; LPDDR4 cost "up sevenfold over the last year" per Raspberry Pi — [BGR](https://www.bgr.com/2150106/raspberry-pi-cost-ram-crisis/); [Gigazine 2026-04-02](https://www.gigazine.net/gsc_news/en/20260402-raspberry-pi-price-increases); [Notebookcheck](https://www.notebookcheck.net/Raspberry-Pi-5-now-costs-up-to-205-due-to-RAM-crisis.1218213.0.html). A further change on 1 Oct 2026 (2GB Pi 5 to $77.50) is reported; 8GB/16GB not confirmed — [OMG! Ubuntu](https://www.omgubuntu.co.uk/2026/10/raspberry-pi-2gb-costs-how-much-now/amp).
- **Intel N100/N150 mini PC**: price conflicting and volatile: Beelink Mini S12 Pro (N100, 16GB, 500GB) quoted at $125-145 in one version of a guide and $250-330 ("DRAM shortage has pushed remaining stock to roughly double") in another; Beelink EQ14 (N150) $169-189 — [computingforgeeks](https://computingforgeeks.com/best-mini-pc-home-assistant/). Trade-off: more headroom (Frigate, local voice, Proxmox VM) but you install HA OS yourself; Green is plug-and-play — same source and [esp32.co.uk](https://esp32.co.uk/?p=991201).

#### Nabu Casa subscription (Home Assistant Cloud, being renamed Home Assistant Link)

- US price: **$6.50/month or $65/year** (sales tax extra); EUR 7.50/75, GBP 6.50/65, CAD 8.70/87; 30-day free trial — [Nabu Casa pricing](https://www.nabucasa.com/pricing/) (search summary). A third-party site quotes about $8/$80 — [tecnoyfoto](https://tecnoyfoto.com/en/home-assistant-cloud-worth-it-2026); treat as unverified/outdated.
- Includes secure remote access, Amazon Alexa and Google Assistant integrations (exposing HA entities to those voice assistants without port forwarding), cloud TTS/STT for Assist, and encrypted cloud backups — [HA Cloud page](https://www.home-assistant.io/cloud/) (search summary); [tbreak](https://tbreak.com/home-assistant-link-replaces-cloud/).
- **Changed Oct 2026:** Nabu Casa announced on 2 Oct 2026 that Home Assistant Cloud is renamed **Home Assistant Link**; features unchanged; name change planned with the December (2026.12) release — [HA blog: "Big Tech ruined the cloud, so we're renaming ours"](https://home-assistant.io/blog/2026/10/02/big-tech-ruined-the-cloud-so-were-renaming-ours/) (search summary); [Nabu Casa news](https://www.nabucasa.com/news/2026-10-02-cloud-becomes-link/); [smarthomeassistent.de](https://www.smarthomeassistent.de/home-assistant-cloud-wird-umbenannt/). Whether pricing changes with the rename: no indication found.

### Inferences

- Option A total cost of entry: about $199 (Green) + $0/year (local only) or + $65/year if the owner wants remote access and "Alexa, turn on X" for HA-only devices. A Pi 5 8GB plus PSU, case and SSD now costs roughly the same or more than a Green, which removes the Pi's old price advantage.
- For a wall tablet on the LAN, Nabu Casa is optional: the dashboard and all local integrations work without it.
- Option A is the fastest path to a working, robust panel and is also the natural first step of Option B (the integrations must be set up in HA either way).
- Monthly releases with breaking-change sections mean some maintenance: a dashboard built from many HACS custom cards depends on each card keeping up with frontend changes.

### Gaps

- Official Nabu Casa US store price page for Green could not be fetched; the $199 figure rests on Nabu Casa's own January 2026 post (via search) plus secondary coverage.
- No primary source checked for whether Mushroom/Bubble Card have known incompatibilities with 2026.x releases.

## Q3. Option B: Home Assistant as hub, with a custom-built front end over HA's WebSocket/REST APIs

### Takeaway

This is a well-trodden, supported path: HA's WebSocket API is the same one the official frontend and mobile apps use, the official `home-assistant-js-websocket` library (TypeScript, zero dependencies, used by HA's own frontend) handles OAuth/PKCE login, token refresh, reconnection and live entity subscriptions, and several React projects (HAKit, Glance, dcy-glow-board, hass-react) show the pattern working on wall tablets. The owner gets full design control and only has to talk to one API.

### Cited Findings

#### WebSocket API (official docs)

- Flow: server sends `auth_required`; client's first message is `auth` with an `access_token`; server replies `auth_ok` or `auth_invalid` (and disconnects). After auth every message carries an integer `id`. `auth_required`/`auth_ok` include `ha_version` — [developers.home-assistant.io WebSocket API (source)](https://raw.githubusercontent.com/home-assistant/developers.home-assistant/master/docs/api/websocket.md).
- Documented commands: `subscribe_events`, `subscribe_trigger` (automation trigger syntax), `call_service`, `get_states`, `ping`/`pong`; optional `supported_features` message with `coalesce_messages` ("sent coalesced in bulk instead of individually") — same source.
- The doc page contains **no statement about stability guarantees or versioning** for third-party clients, and does not document `subscribe_entities` (the compact entity-state subscription the library uses) — same source.

#### Auth

- OAuth2 with IndieAuth-style clients: `client_id` is the app's website URL, `redirect_uri` must share host/port; `/auth/authorize` -> code -> `/auth/token` returns a short-lived access token (`expires_in`) plus a refresh token; `/auth/revoke` revokes a refresh token and all access tokens it granted — [HA Auth API (source)](https://raw.githubusercontent.com/home-assistant/developers.home-assistant/master/docs/auth_api.md).
- "Long-lived access tokens are valid for 10 years"; created on the user profile page or via WebSocket `auth/long_lived_access_token` (`client_name`, `lifespan`); "The access token string is not saved in Home Assistant" — same source.

#### home-assistant-js-websocket (official client library)

- TypeScript, 414 stars, pushed 2026-10-12, "0 dependencies" — [GitHub](https://github.com/home-assistant/home-assistant-js-websocket).
- `getAuth({hassUrl})` handles redirect to the HA login and token fetch, using S256 PKCE in a secure context; README: "If you are in a browser, you should prefer to use the `getAuth()` flow" over long-lived tokens; `createConnection`, `subscribeEntities` (callback on initial load and every state change), `callService(connection, domain, service, serviceData?, target?, returnResponse?)`; Node.js needs a WebSocket polyfill (`globalThis.WebSocket = require("ws")`) — same source.
- HA's own frontend depends on `home-assistant-js-websocket` 9.7.1 — [home-assistant/frontend package.json](https://raw.githubusercontent.com/home-assistant/frontend/dev/package.json).

#### Existing custom front ends on HA

- **HA Component Kit (@hakit/core, @hakit/components)**: React + TypeScript + Vite; `npm create hakit@latest` scaffolds a project; wraps the official WebSocket API; can be served from any web server or through an HA add-on that adds a sidebar link; 1.3k stars, last push 2026-03-02 (about 7 months without commits) — [shannonhochkins/ha-component-kit](https://github.com/shannonhochkins/ha-component-kit). Its predecessor ha-dashboard is unmaintained in favour of @hakit — [shannonhochkins/ha-dashboard](https://github.com/shannonhochkins/ha-dashboard).
- **Glance**: React 19 + TypeScript + Vite dashboard talking directly to HA's WebSocket API, runs as an HA add-on, "works great with Fully Kiosk on wall tablets"; beta — [HA community thread](https://community.home-assistant.io/t/glance-a-polished-fully-editable-react-dashboard-for-home-assistant-runs-as-an-add-on/1012606).
- **Dcy Glow Board**: React 19 + TS + Vite, connects straight to HA WebSocket with no backend, MIT — [HACF forum](https://forum.hacf.fr/t/dcy-glow-board-un-dashboard-home-assistant-fait-maison-react-sans-lovelace-sans-yaml/82924).
- **hass-react**: headless React hooks/components with typed controls and a mock mode for developing without a live HA — [dlwiest/hass-react](https://github.com/dlwiest/hass-react).
- **HA Studio Pro**: native iPad/Android drag-and-drop dashboard editor for HA — [HA community thread](https://community.home-assistant.io/t/ha-studio-pro-a-native-drag-and-drop-dashboard-editor-for-home-assistant-ipad-android/1015365).
- Other community examples: "Clean Light React Dashboard" built because Lovelace "wasn't customizable enough" — [HA community](https://community.home-assistant.io/t/clean-light-react-dashboard/358815); a custom panel inside HA gets the `hass` object and avoids a separate socket — [HA community "React dashboard"](https://community.home-assistant.io/t/react-dashboard/208107).

### Inferences

- Stability: there is no written versioning contract, but because the official frontend and companion apps use the same API and the same library, breaking changes to core messages (`auth`, `call_service`, `subscribe_events`, entity state shape) would break HA's own UI; in practice the API is stable, while individual integration entity attributes and service names change more often (watch the monthly "backward-incompatible changes" section).
- Recommended deployment shape for the owner: (1) a dedicated HA user for the tablet; (2) the page served from the same origin as HA (add-on or `/local/` www folder) or from a LAN server with HA's `cors_allowed_origins` set; (3) `getAuth()` login once on the tablet so a revocable refresh token is stored, rather than a 10-year long-lived token baked into the bundle.
- Things the custom page must re-create that Lovelace gives for free: entity pickers/configuration UI, more-info dialogs, media browsing, camera streams, and handling of entity availability/unavailable states. A spec-driven project can limit scope to the owner's fixed set of entities, which makes this manageable.
- HAKit's slowing commit activity suggests building on `home-assistant-js-websocket` directly (or using HAKit only as a reference) to avoid depending on a single-maintainer UI kit.

### Gaps

- TileBoard (resoai/TileBoard) did not appear in the GitHub search results; its current maintenance status is unverified.
- No official HA developer statement found on WebSocket API deprecation policy for third-party clients.
- Whether HA non-admin users can be limited to specific entities/services (useful for a tablet user) was not verified.

## Q4. Option C: fully custom backend without Home Assistant

### Takeaway

Feasible but by far the most effort: the owner would re-implement, per service, OAuth flows and token storage (Google, Nest SDM, Amazon), LAN discovery and push/event handling (Hue, Sonos), reconnection and rate limiting, all of which HA's integrations already do and keep updated monthly. The library ecosystem is strongest in Python (HA's own libraries), reasonable in Node/TypeScript, and thinnest in Java (Sonos especially). Alexa has no official API for controlling Echo devices from a third-party app, so a custom backend would have to use the same reverse-engineered approach as HA's Alexa integrations.

### Cited Findings

**What HA uses per service (the libraries a custom backend would replace or reuse)** — from core `manifest.json` files:

- Sonos: `local_push`, requires `soco==0.31.5` and `sonos-websocket==0.2.0` — [sonos/manifest.json](https://raw.githubusercontent.com/home-assistant/core/dev/homeassistant/components/sonos/manifest.json).
- Hue: `local_push`, `aiohue==4.9.0` — [hue/manifest.json](https://raw.githubusercontent.com/home-assistant/core/dev/homeassistant/components/hue/manifest.json).
- Nest: `cloud_push`, `google-nest-sdm==9.2.1`, depends on `application_credentials` (user-supplied OAuth client) and `ffmpeg` — [nest/manifest.json](https://raw.githubusercontent.com/home-assistant/core/dev/homeassistant/components/nest/manifest.json).
- Google Calendar: `cloud_polling`, `gcal-sync==9.2.0`, `ical==14.3.0` — [google/manifest.json](https://raw.githubusercontent.com/home-assistant/core/dev/homeassistant/components/google/manifest.json).
- Google Tasks: `cloud_polling`, `google-api-python-client` — [google_tasks/manifest.json](https://raw.githubusercontent.com/home-assistant/core/dev/homeassistant/components/google_tasks/manifest.json).

#### Library landscape by language (GitHub, 2026-10-17)

- Python: SoCo (Sonos) 1.6k stars, pushed 2026-10-15 — [SoCo/SoCo](https://github.com/SoCo/SoCo); aiohue 75 stars, pushed 2026-10-14 — [home-assistant-libs/aiohue](https://github.com/home-assistant-libs/aiohue); python-google-nest-sdm 72 stars, pushed 2026-10-16 — [allenporter/python-google-nest-sdm](https://github.com/allenporter/python-google-nest-sdm). These are the libraries HA itself ships, so they are actively maintained.
- Node/TypeScript: googleapis (official, TypeScript) 12.3k stars, pushed 2026-10-14 — [googleapis/google-api-nodejs-client](https://github.com/googleapis/google-api-nodejs-client); node-hue-api 1.2k stars, pushed 2026-07-12 — [peter-murray/node-hue-api](https://github.com/peter-murray/node-hue-api); @svrooij/sonos (node-sonos-ts) 111 stars, pushed 2026-07-08 — [svrooij/node-sonos-ts](https://github.com/svrooij/node-sonos-ts); node-sonos-http-api 2.0k stars but last push 2025-03-22 — [jishi/node-sonos-http-api](https://github.com/jishi/node-sonos-http-api).
- Java: Google APIs Client Library for Java 1.5k stars, pushed 2026-10-14 — [googleapis/google-api-java-client](https://github.com/googleapis/google-api-java-client); yetanotherhueapi (Hue) 70 stars, pushed 2026-07-24 — [ZeroOne3010/yetanotherhueapi](https://github.com/ZeroOne3010/yetanotherhueapi); sonos-controller (Java Sonos) **archived**, last push 2020-05-24 — [vmichalak/sonos-controller](https://github.com/vmichalak/sonos-controller).
- Alexa: HA's core Alexa Devices integration is `Cloud Polling` against the Amazon account and requires authenticator-app MFA — [alexa_devices docs (source)](https://raw.githubusercontent.com/home-assistant/home-assistant.io/current/source/_integrations/alexa_devices.markdown); the HACS Alexa Media Player warns "Amazon may cut off access at anytime" — [alandtse/alexa_media_player](https://github.com/alandtse/alexa_media_player).
- Google Keep: no official consumer API path used by HA; the community integration uses gkeepapi with a "master token" (`aas_et/...`) that "grants the account's full Keep access", recommends a dedicated Google account, polls every 15 minutes, and is "not created, endorsed, or supported by Google" — [watkins-matt/home-assistant-google-keep-sync](https://github.com/watkins-matt/home-assistant-google-keep-sync).

### Inferences

- Rough effort per service for Option C (owner's own estimate needed; these are judgments, not sourced figures): Google Calendar/Tasks via official SDKs: small (OAuth code flow + refresh token store + polling); Nest SDM: medium (Device Access project, OAuth, Pub/Sub for push events); Hue: small-medium (bridge pairing, local HTTPS with self-signed cert, event stream); Sonos: medium-large in Java (no maintained library; UPnP/SOAP + event subscriptions), smaller in Node or Python; Alexa: large and fragile (reverse-engineered, MFA, no official API); assorted Wi-Fi bulbs: one integration per brand. HA covers all of these out of the box and absorbs upstream API breakage each month.
- Option C makes sense only if the owner wants the integration code itself to be the hobby, or wants to avoid running HA at all. Even then, a sensible variant is to embed HA's Python libraries (SoCo, aiohue, google-nest-sdm) in a Python service rather than rewrite them in Java.
- Polling vs push: Hue, Sonos (local push) and Nest (cloud push via Pub/Sub) offer push; Google Calendar/Tasks and weather are polling. A custom backend needs both models plus a push channel to the tablet (WebSocket or SSE), which is exactly what HA's WebSocket API already provides.

### Gaps

- No verified Java library for the Nest SDM API specifically (the generated `google-api-services-smartdevicemanagement` artifact is assumed to exist but was not checked).
- No source found quantifying developer effort for each integration; the effort ranking above is inference.

## Q5. Option D: off-the-shelf products and alternatives

### Takeaway

No off-the-shelf product covers the whole list (Alexa + Hue + Wi-Fi bulbs + Sonos + Nest + Google Calendar + Google Keep + weather) with a self-designed page. Echo Show/Echo Hub cover Alexa-side devices, calendar and weather with almost no layout control; Nest Hub is aging (last hardware 2021); DAKboard and MagicMirror² are display/calendar boards with little or no device control; Homey Pro and Hubitat are capable hubs with their own dashboards but cost more than HA Green and have smaller ecosystems; openHAB is the Java-based open-source alternative to HA; Homarr and Dashy are homelab service dashboards, not home-control panels.

### Cited Findings

- **Amazon Echo Hub**: 8-inch wall panel with customizable dashboard, Matter/Zigbee/Thread/Sidewalk, no camera; list $179.99, deals at $119.99-129.99; marketed as "Designed for Alexa+" — [Thurrott](https://www.thurrott.com/?p=298199); [Tom's Guide](https://tomsguide.com/home/smart-home/amazons-echo-hub-is-my-favorite-smart-home-controller-and-it-just-hit-its-lowest-price-ever); [Slickdeals Dec 2025](https://slickdeals.net/f/18948961-amazon-echo-hub-8-smart-home-control-panel-with-alexa-early-access-119-99).
- **Amazon Echo Show 15**: launched $249.99 (2021); widgets for calendars, to-do lists, reminders; widgets mostly one size and not resizable, customization limited; one Feb 2026 listing at $179.95; a second generation is reported — [TechRadar review](https://www.techradar.com/uk/reviews/amazon-echo-show-15); [Tom's Guide opinion](https://www.tomsguide.com/opinion/amazon-is-wasting-an-opportunity-with-the-echo-show-15); [shopsavvy](https://shopsavvy.com/reviews-tldr/echo-show-15).
- **Google Nest Hub**: newest model is gen 2 (2021); Gemini for Home is replacing Google Assistant on Nest Hub 1st/2nd gen and Nest Hub Max (Gemini Live added to 1st-gen Nest Hub per a 23 July 2026 release note); a "Google Home Display" is only rumoured (references in the Google Home app, May 2026); Google Home Speaker announced at $99 — [Tech Advisor (May 2026)](https://www.techadvisor.com/article/3139852/google-home-display-could-be-a-gemini-powered-smart-screen.html); [TuttoAndroid (July 2026)](https://www.tuttoandroid.net/news/2026/07/24/gemini-live-google-home-mini-nest-hub-prima-gen-1164239/); [Google blog: Gemini for Home](https://blog.google/intl/en-au/products/devices-services/try-the-new-gemini-for-home-voice-assistant/).
- **Homey Pro (Athom/LG)**: **changed June 2026:** Homey Pro $399 -> $449, Homey Pro mini $199 -> $249 (effective 1 June 2026; memory/storage costs, Raspberry Pi component) — [Homey: price update June 2026](https://homey.app/en-ca/news/homey-pro-price-update-june-2026/); [Android Authority](https://androidauthority.com/homey-price-hike-3668371/). Homey Dashboards: multiple dashboards, "turn any tablet into a smart home controller with Kiosk Mode"; app widgets only on Homey Pro, not mini — [Homey Dashboards](https://homey.app/en-ca/features/dashboards/).
- **Hubitat Elevation C-8 Pro**: $179.95 (confirmed from manufacturer 2026-07-16 by a buyer's guide; earlier deal $149.95) — [PromptQuorum](https://www.promptquorum.com/smart-home/best-smart-home-hubs-2027); [Slickdeals](https://slickdeals.net/f/19389747-hubitat-elevation-c-8-pro-149-95).
- **openHAB** (Java, open source): 5.0 released 21 July 2025, requires Java 21, drops 32-bit — [openHAB 5.0 release](https://openhab.org/blog/2025-07-21-openhab-5-0-release.html); 5.1 (22 Dec 2025) rebuilt Main UI on Vue 3 + Framework7 v7 + Vite, introduced TypeScript, added a HomeKit binding — [openHAB 5.1 release](https://www.openhab.org/blog/2025-12-22-openhab-5-1-release); 5.2 is out and backward compatible with 5.x — [newreleases.io](https://newreleases.io/project/github/openhab/openhab-distro/release/5.2.0). openHAB web UI repo: 259 stars, Vue, pushed 2026-10-13 — [openhab/openhab-webui](https://github.com/openhab/openhab-webui).
- **DAKboard**: cloud-hosted display boards (calendar, weather, photos); official pricing page shows "Pro" at $35/month or $30/month billed annually (5 custom screens, white-labeling, SLA); Essential and Plus tiers exist (30-day trial) but prices not confirmed — [DAKboard pricing](https://dakboard.com/pricing) (search summary, snippet partly garbled); [subscribed.fyi](https://subscribed.fyi/dakboard/reviews/).
- **MagicMirror²**: open-source modular display platform (JavaScript/Electron), 24.1k stars, pushed 2026-10-14 — [MagicMirrorOrg/MagicMirror](https://github.com/MagicMirrorOrg/MagicMirror).
- **Homarr** ("80+ integrations", TypeScript) 5.1k stars, pushed 2026-10-17 — [homarr-labs/homarr](https://github.com/homarr-labs/homarr); **Dashy** (Vue, "self-hostable personal dashboard" with status checks) 26.7k stars, pushed 2026-10-16 — [lissy93/dashy](https://github.com/lissy93/dashy).
- **Dedicated wall panels (2026)**: vendor claims of Matter-capable panels such as Aqara Panel Hub S1 and Sonoff NSPanel Pro, and HA-oriented panels like LinknLink iSG Display Max — [LinknLink blog](https://www.linknlink.com/blogs/news/isg-display-max-ultimate-home-assistant-control-hub-2026) (vendor source, unverified). A repurposed Fire HD 8 + mount + kiosk browser is quoted at under about $65 — [joinhomeshift](https://joinhomeshift.com/home-assistant-tablet/).

### Inferences

Coverage of the owner's list (judgment based on the findings above and general product scope; verify per product):

| Product | Alexa/Echo | Hue / Wi-Fi bulbs | Sonos | Nest thermostat | Google Calendar | Google Keep | Weather | Own page design | Price (US) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Echo Hub / Echo Show | native | via Alexa | via Alexa skill | via Alexa skill | yes (linked) | no | yes | none (widgets) | $120-250 |
| Nest Hub | no | via Google Home | limited | native | native | no | yes | none | aging (2021 hw) |
| DAKboard | no | no | no | no | yes | no | yes | layout editor only | subscription |
| MagicMirror² | via modules | via modules | via modules | via modules | yes | modules | yes | full (JS modules) | free + Pi |
| Homey Pro | apps | yes | yes | yes | apps | no | yes | widget dashboards | $449 |
| Hubitat C-8 Pro | limited | yes | yes | community | community | no | community | dashboards | $179.95 |
| openHAB | binding | yes | yes | binding | binding | no | yes | Main UI / custom | free + hw |
| Home Assistant | integrations | yes | yes | yes | yes | HACS only | yes | full (Option B) | $199 Green |
| Homarr / Dashy | no | no | no | no | partial | no | partial | n/a (homelab) | free |

- MagicMirror² is the closest "design your own page" alternative without HA, but each device needs a community module (effectively Option C in JavaScript with a plugin framework).
- Echo Hub is a strong cheap fallback for Alexa-only devices but gives no layout control and no Keep.

### Gaps

- SmartThings: no 2026 sources found on a SmartThings wall dashboard product or its coverage of Hue/Sonos/Nest; not assessed.
- Hubitat Easy Dashboard features/cost not verified; Homey Nest/Sonos app quality not verified.
- DAKboard Essential/Plus prices and whether DAKboard still offers a free tier: not verified.
- Echo Show 15 current generation and Alexa+ pricing for Prime/non-Prime US users not verified in this session.
- ImmichFrame-like photo frames and other 2025-2026 entrants were not researched for lack of budget.

## Q6. Hybrid: what HA's integrations cover from the owner's list (so the custom app talks to one API)

### Takeaway

HA covers nearly the whole list with core integrations: Hue and Sonos (local push), Nest (cloud push via SDM), Google Calendar and Google Tasks (cloud polling), weather (NWS for the US, cloud polling), and Echo devices via the core Alexa Devices integration (since 2025.6, platinum quality). The two weak spots are Google Keep (only an unofficial HACS integration using a reverse-engineered API) and Wi-Fi bulbs/plugs that exist only in the Alexa app (neither Alexa integration exposes Alexa-side smart-home devices as HA lights; options are Alexa routines triggered from HA or adding each brand's own HA integration).

### Cited Findings

- **Hue**: core, `local_push` — [hue/manifest.json](https://raw.githubusercontent.com/home-assistant/core/dev/homeassistant/components/hue/manifest.json).
- **Sonos**: core, `local_push`, quality scale bronze — [sonos/manifest.json](https://raw.githubusercontent.com/home-assistant/core/dev/homeassistant/components/sonos/manifest.json).
- **Nest thermostat**: core, `cloud_push`, uses user-provided application credentials — [nest/manifest.json](https://raw.githubusercontent.com/home-assistant/core/dev/homeassistant/components/nest/manifest.json).
- **Google Calendar**: core "Google Calendar", `cloud_polling` — [google/manifest.json](https://raw.githubusercontent.com/home-assistant/core/dev/homeassistant/components/google/manifest.json).
- **Google Tasks**: core, `cloud_polling`; adds a to-do entity per task list (create/update/delete items); needs Google developer credentials like Nest; the Tasks API exposes neither due times nor recurrence — [google_tasks/manifest.json](https://raw.githubusercontent.com/home-assistant/core/dev/homeassistant/components/google_tasks/manifest.json); [HA Google Tasks docs](https://home-assistant.io/integrations/google_tasks) (search summary).
- **Weather (US)**: National Weather Service integration, core, `cloud_polling`, `pynws` — [nws/manifest.json](https://raw.githubusercontent.com/home-assistant/core/dev/homeassistant/components/nws/manifest.json).
- **Alexa/Echo, core "Alexa Devices"**: `ha_release: 2025.6`, `Cloud Polling`, quality scale **platinum**; platforms binary_sensor, button, event, media_player, notify, select, sensor, switch, todo; entities include Speak/Announce notify, alarm/timer/reminder sensors, buttons "for running routines" and restarting devices, Do-not-disturb switch, Drop In select, to-do lists (shopping, to-do, custom), an event entity of voice interactions; requires MFA with an authenticator app (SMS not supported); limitations include rate-limit warnings and no music via SSML. The docs do not mention controlling smart-home devices (lights/plugs) configured in the Alexa app — [alexa_devices docs (source)](https://raw.githubusercontent.com/home-assistant/home-assistant.io/current/source/_integrations/alexa_devices.markdown). A 2026 core bug with media pause calls was reported in the community — [simon42 community](https://community.simon42.com/t/alexa-devices-stoerung/88338).
- **Alexa Media Player (HACS)**: ~2.0k stars, pushed 2026-10-16; "Allows for control of Amazon Echo products as home assistant media devices"; "Amazon may cut off access at anytime" — [alandtse/alexa_media_player](https://github.com/alandtse/alexa_media_player).
- **Exposing HA devices to Alexa** (the other direction, so "Alexa, turn on the Hue lamp" keeps working): via Home Assistant Cloud/Link, a manual Smart Home Skill, Emulated Hue, or a Matter bridge — [search summary of HA Alexa docs and Sonoff guide](https://sonoff.tech/blogs/news/home-assistant-alexa-integration-via-matter-bridge).
- **Google Keep**: no core integration; HACS "Google Keep Sync" (126 stars, pushed 2026-10-11) gives bidirectional to-do entities using a master token, 15-minute polling, unofficial — [watkins-matt/home-assistant-google-keep-sync](https://github.com/watkins-matt/home-assistant-google-keep-sync); users report token-generation trouble in 2026 — [HA community](https://community.home-assistant.io/t/trouble-generating-google-keep-sync-token/1022890).

### Inferences

- With HA as hub, the custom page needs exactly one integration: HA WebSocket. Every device and Google list appears as an entity (`light.*`, `media_player.*`, `climate.*`, `calendar.*`, `todo.*`, `weather.*`) with uniform `call_service` actions. This is the main architectural argument for Option B.
- Alexa-only Wi-Fi bulbs/plugs: best fix is to add them to HA through the bulb brand's own integration (device researcher to confirm per brand). Fallback: create Alexa routines ("lamp on"/"lamp off") and trigger them from HA via the Alexa Devices routine buttons.
- Google Keep is the riskiest item on the owner's list under any architecture; the spec should treat Keep as optional or consider moving lists to Google Tasks or the Alexa shopping/to-do lists (both exposed as HA `todo` entities).

### Gaps

- Whether Alexa can share Matter devices it controls to HA (Matter multi-admin from Alexa) was not verified.
- Met.no (HA's default onboarding weather) IoT class and US suitability not verified this session.
- HA's Google Calendar integration write support (create events) not verified here.

## Q7. Implications for the implementation-language choice

### Takeaway

Whatever the backend language, the page on the tablet is browser code, so TypeScript (or JavaScript) is effectively required for the front end. The HA frontend ecosystem is TypeScript plus Lit web components; custom-dashboard projects are mostly React + TypeScript + Vite; even openHAB, the Java hub, moved its UI to Vue 3 + TypeScript in 2025. Under Option B no backend is strictly needed; a Java backend would only be a thin backend-for-frontend (BFF) that holds tokens and proxies HA, which adds a second language without removing the TypeScript front end.

### Cited Findings

- HA frontend: TypeScript (5.7k stars) built on Lit 3.3.3 (`lit`, `@lit/context`, `@lit/task`, `@lit-labs/virtualizer`), `home-assistant-js-websocket` 9.7.1, Material web components and MDI icons, built with rspack (Vite in dev deps); no React or Vue — [home-assistant/frontend](https://github.com/home-assistant/frontend); [frontend package.json](https://raw.githubusercontent.com/home-assistant/frontend/dev/package.json).
- HA core and its device libraries are Python — [home-assistant/core](https://github.com/home-assistant/core); [SoCo](https://github.com/SoCo/SoCo); [aiohue](https://github.com/home-assistant-libs/aiohue).
- Popular custom cards are TypeScript or JavaScript web components (Mushroom TS, Bubble Card JS, browser_mod TS, kiosk-mode TS) — [lovelace-mushroom](https://github.com/piitaya/lovelace-mushroom); [Bubble-Card](https://github.com/Clooos/Bubble-Card); [hass-browser_mod](https://github.com/thomasloven/hass-browser_mod); [kiosk-mode](https://github.com/NemesisRE/kiosk-mode).
- Custom HA front ends: HAKit, Glance, dcy-glow-board, hass-react are React + TypeScript (+ Vite) — see Q3 sources ([ha-component-kit](https://github.com/shannonhochkins/ha-component-kit), [Glance thread](https://community.home-assistant.io/t/glance-a-polished-fully-editable-react-dashboard-for-home-assistant-runs-as-an-add-on/1012606), [hass-react](https://github.com/dlwiest/hass-react)).
- `home-assistant-js-websocket` works in Node.js with a `ws` polyfill, so a TypeScript backend can reuse the same client library as the browser — [home-assistant-js-websocket](https://github.com/home-assistant/home-assistant-js-websocket).
- openHAB (Java 21 runtime) rebuilt its Main UI on Vue 3 + Framework7 + Vite with TypeScript in 5.1 (Dec 2025) — [openHAB 5.1 release](https://www.openhab.org/blog/2025-12-22-openhab-5-1-release).
- Java library support for the owner's devices is thinner (Java Sonos library archived since 2020; small Hue library) while Google's official Java client is active — see Q4 sources.

### Inferences

- **Option A**: no language choice needed (YAML + optional custom cards in TS/Lit).
- **Option B, frontend-only**: TypeScript single-language project (e.g., Lit or React + Vite + home-assistant-js-websocket), served from HA. Lowest moving parts; recommended default.
- **Option B with BFF**: if the owner wants Java (consistency with their other projects), a small Spring Boot or Quarkus service could: hold the HA token server-side, connect to HA's WebSocket with a standard Java WebSocket client, expose a narrow REST + SSE/WebSocket API tailored to the page, add features HA lacks (e.g., a Keep adapter), and serve the static page (same origin, no CORS). The tablet page is still TypeScript. A TypeScript-on-Node BFF would instead share types and the official HA client library with the front end.
- **Option C**: language choice matters most here; Python has the best device libraries (HA's own), Node is adequate, Java is weakest for Sonos/Alexa.
- Net: the language decision for this project mainly affects an optional BFF; the front-end language is effectively fixed to TypeScript. That is a reasonable input to the project's ADR on implementation language.

### Gaps

- No maintained, widely used Java client for HA's WebSocket API was found or checked (not searched for lack of budget).

## Q8. Overall comparison: which option suits a hobbyist who wants to design their own page

### Takeaway

Recommended: **Option B (HA as integration hub + own TypeScript front end)**, built on top of a working Option A install. HA absorbs all device/OAuth/discovery work for about $199 of hardware and $0-65/year; the owner designs the page freely against one stable, well-documented API. Option C costs many times the effort for the same result, and off-the-shelf products (Option D) do not cover the list or allow real design control.

### Cited Findings

- Hardware/subscription costs: HA Green $199 ([Nabu Casa](https://www.nabucasa.com/news/2026-01-08-green-pricing-change/), [Notebookcheck](https://www.notebookcheck.net/Rising-costs-in-the-Smart-Home-The-impact-of-the-chip-and-RAM-shortage.1338441.0.html)); Pi 5 8GB about $175 board-only ([BGR](https://www.bgr.com/2150106/raspberry-pi-cost-ram-crisis/)); N100 mini PC $125-330 depending on stock ([computingforgeeks](https://computingforgeeks.com/best-mini-pc-home-assistant/)); Nabu Casa $65/year optional ([Nabu Casa pricing](https://www.nabucasa.com/pricing/)); Homey Pro $449 ([Homey](https://homey.app/en-ca/news/homey-pro-price-update-june-2026/)); Hubitat C-8 Pro $179.95 ([PromptQuorum](https://www.promptquorum.com/smart-home/best-smart-home-hubs-2027)); Echo Hub $179.99 list ([Thurrott](https://www.thurrott.com/?p=298199)).
- HA integration coverage of the list and the official WebSocket client are documented in Q3 and Q6.

### Inferences

Summary comparison (judgments drawn from the findings above):

| | A: HA + Lovelace | B: HA + custom front end | C: fully custom backend | D: off-the-shelf |
| --- | --- | --- | --- | --- |
| Device coverage of owner's list | high (Keep via HACS only) | same as A | whatever the owner builds | partial |
| Design control | medium (cards, CSS, YAML) | full | full | low |
| Effort to first working panel | days | weeks | months | hours |
| Ongoing maintenance | HA updates + HACS cards | HA updates + own code (one API) | every upstream API change | vendor-driven |
| Secrets on tablet | one HA token | one HA token (or none with BFF) | none (backend holds all) | vendor account |
| Upfront cost | ~$199 Green (or existing PC) | same | similar hardware | $120-450 + subscriptions |
| Fit for spec-driven hobby project | low (config, not code) | high | high but huge scope | none |

- Suggested phasing: (1) install HA (Green or N100), add Hue, Sonos, Nest, Google Calendar, Google Tasks, NWS, Alexa Devices, brand integrations for Wi-Fi bulbs; (2) make a quick Lovelace tablet view as a fallback and to learn entity IDs; (3) spec and build the custom page against HA's WebSocket API, served from HA; (4) add a BFF only if a requirement needs it (Keep, server-side token, aggregated data).
- Risk register for the report: Alexa (unofficial, MFA, Amazon may block), Google Keep (unofficial master token), Nest SDM (Google policy changes), HA monthly breaking changes, and the 2025-2026 RAM-driven hardware price increases.

### Gaps

- Effort estimates in the table are qualitative judgments, not sourced measurements.
- Prices are volatile in 2026 (several increases within the year); all hardware prices should be rechecked at purchase time.
