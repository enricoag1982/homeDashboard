---
status: concluded
date: 2026-10-17
authors: [Claude Code]
related: [2026-10-17-tablet-home-dashboard-options]
---

# Lights (Philips Hue, Wi-Fi brands, Matter) and Google Nest / temperature for a self-built home dashboard (state as of October 2026)

Research notes. Owner context: a household in the United States. Lights: Philips Hue with a Hue Bridge, plus Wi-Fi bulbs and plugs of unknown brands, some set up only in the Alexa app. Thermostat: Google Nest, generation unknown. Research date: 2026-10-17.

**How these notes were sourced.** In this research environment, direct page fetches failed for developers.google.com, developers.meethue.com, philips-hue.com, home-assistant.io, developer.govee.com, api.developer.lifx.com and shelly-api-docs.shelly.cloud (DNS or proxy refusal). So:

- **Primary sources read in full:**
  - Home Assistant (HA) integration docs, read from their GitHub source (`github.com/home-assistant/home-assistant.io`, branch `current`). The canonical home-assistant.io URLs are cited below.
  - HA core source files and library READMEs on GitHub.
  - Package release data from PyPI and npm, fetched 2026-10-17.
- **Search-engine extracts only:** findings for official Google, Signify, Govee, LIFX and Shelly pages come from search extracts and were not fetched directly. They are marked "(search extract)". Re-check them before relying on exact numbers.

---

## Q1. Philips Hue: local CLIP API v2, Remote API, Hue Bridge Pro, 2025-2026 changes, libraries, HA integration

### Takeaway

The square Hue Bridge (v2) and the new Hue Bridge Pro both expose the local CLIP API v2 over HTTPS, with Server-Sent Events push. This is the best-supported local light API the owner has. Things to know about the Bridge Pro (2025):

- It is HTTPS-only.
- It uses certificates from Signify's own CA instead of self-signed ones.
- It drops UPnP discovery in favor of mDNS.

A browser page on a wall tablet therefore should not talk to the bridge directly. A small backend or Home Assistant in between is the practical design. HA's Hue integration is local push, current and well maintained.

### Cited Findings

#### Local API (CLIP v1/v2), event stream

- Bridges are reached through the CLIP API in two versions, v1 and v2. Signify has said new features (for example dynamic scenes) will appear only on API v2, and API v1 will eventually be removed. API v2 supports Server-Sent Events, so status updates arrive faster. API v2 is not supported by round v1 bridges, nor by square bridges on firmware below 1948086000 — [openHAB Hue binding README (GitHub)](https://github.com/openhab/openhab-addons/blob/main/bundles/org.openhab.binding.hue/README.md).
- Authorization works by pressing the bridge's link button so the client can register — [openHAB Hue binding README](https://github.com/openhab/openhab-addons/blob/main/bundles/org.openhab.binding.hue/README.md). Third-party clients describe the same flow ("press the link button on the bridge first", then use bridge IP plus application key) — [HueApi on NuGet (search extract)](https://www.nuget.org/packages/HueApi/).
- The CLIP v2 event stream lives at `/eventstream/clip/v2`. A community wiki says the bridge allows only about 3 concurrent API connections. This is a single community claim and was not verified against Signify — [Domoticz wiki (search extract)](https://wiki.domoticz.com/Philips_Hue_Lights).
- HA docs: "V2 Hue bridges (square shape) push state changes to Home Assistant instantly over their local event stream. V1 Hue bridges (round shape) are polled." The round V1 bridge "is now end of life and no longer supported by Signify" — [HA Philips Hue integration](https://www.home-assistant.io/integrations/hue/).
- HA docs: "The Hue API limits each device to one event per second", so button events are rate-limited to one per second — [HA Philips Hue integration](https://www.home-assistant.io/integrations/hue/).
- HA recommends Hue scenes over per-light commands for multi-light changes: "If you control multiple lights individually... each command is sent to each light one by one. A Hue scene sends commands to all lights at once in an optimized way" — [HA Philips Hue integration](https://www.home-assistant.io/integrations/hue/).
- OpenHue publishes an open-source OpenAPI specification of the CLIP v2 API, which can be used to generate clients in any language, including Java and TypeScript — [openhue/openhue-api README](https://github.com/openhue/openhue-api).

#### Hue Bridge Pro (2025)

- Capacity: 150+ lights, 50+ accessories, up to 500 scenes. That is about three times the standard bridge — [Philips Hue UK product page (search extract)](https://www.philips-hue.com/content/hue/gb/en/products/all-products/product-page.HUE_BRIDGE_PRO.html). The old bridge's official limit was 50 lights — [MightyGadget review (search extract)](https://mightygadget.com/philips-hue-bridge-pro-review/).
- Hardware: quad-core 1.7 GHz "Hue Chip Pro" and 8 GB RAM — [MightyGadget review](https://mightygadget.com/philips-hue-bridge-pro-review/). It adds Wi-Fi alongside Ethernet and takes USB-C power — [Hueblog, 2025-08-14](https://hueblog.com/2025/08/14/hue-bridge-pro-usb-c-wi-fi-and-much-more-power/).
- MotionAware (Hue bulbs used as motion sensors) is exclusive to the Bridge Pro — [MightyGadget review](https://mightygadget.com/philips-hue-bridge-pro-review/). MotionAware is exposed only through API v2 — [homebridge-hue issue #1219](https://github.com/ebaauw/homebridge-hue/issues/1219).
- Price: £89.99 in the UK, AUD 178-190 in Australia. No US price was found — [Philips Hue UK (search extract)](https://www.philips-hue.com/content/hue/gb/en/products/all-products/product-page.HUE_BRIDGE_PRO.html); [Reduction Revolution AU](https://reductionrevolution.com.au/products/hue-bridge-pro).
- Launch date conflicts between sources:
  - A Hueblog FAQ dated 2025-09-04 and a homebridge-hue issue opened 2025-09-08 show the Bridge Pro existed by early September 2025 — [Hueblog Bridge Pro FAQ](https://hueblog.com/2025/09/04/hue-bridge-pro-almost-all-the-answers-to-the-most-important-questions/); [homebridge-hue #1219](https://github.com/ebaauw/homebridge-hue/issues/1219).
  - A Philips release-notes extract names Oct 22, 2025 as a release date — [Hue Bridge Pro release notes (search extract)](https://dev.philips-hue.com/fr-ca/support/release-notes/bridge-pro).
- API and security changes on the Bridge Pro (developer-reported, Sep-Oct 2025; Signify statements are quoted secondhand) — [homebridge-hue issue #1219](https://github.com/ebaauw/homebridge-hue/issues/1219):
  - The Bridge Pro supports only HTTPS. Signify is phasing out plain HTTP, and gen-2 (square) bridges will drop it in a future firmware.
  - Signify replaced self-signed certificates with certificates from its own CA. Clients should stop pinning self-signed certificates.
  - The event stream (API v2) works only over HTTPS.
  - The Bridge Pro does not announce itself over UPnP and no longer serves `description.xml`. Local discovery is by mDNS. The MeetHue discovery server returns port 443.
  - Model ID `BSB003`, API version 1.73.0. API v1 still works over HTTPS (homebridge-hue, a v1 client, works with it).
  - Application keys can be reused after migration by changing the bridge ID, or created again with the link button.
- Other users report the same: "Bridge Pro now enforces https (http redirects to https)" for v1 use, and some third-party (KNX, Loxone) integrations broke — [Hueblog Bridge Pro FAQ comments](https://hueblog.com/2025/09/04/hue-bridge-pro-almost-all-the-answers-to-the-most-important-questions/); [LoxForum thread](https://www.loxforum.com/forum/hardware-zubehör-sensorik/468535-hue-bridge-pro); [Hubitat community](https://community.hubitat.com/t/new-philips-hue-bridge-pro-and-other-stuff/156068?page=4).
- Migration from the old bridge is supported. Hue app 5.57.0 added a multi-bridge migration tool — [Notebookcheck](https://www.notebookcheck.net/Philips-Hue-rolling-out-new-migration-tool-for-multiple-Hue-Bridges.1182261.0.html).

#### Hue account requirement

- In 2023 Signify announced that a Hue account would be required for the Hue app. It said physical access to the bridge (the button press) "is no longer sufficient" — [Hueblog 2023-09-21](https://hueblog.com/2023/09/21/philips-hue-soon-only-usable-with-account-2/); [TechTimes](https://www.techtimes.com/articles/296944/20230928/philips-hue-users-will-soon-required-sign-up-accountbut-customers.htm).
- Born's Tech, citing The Verge, reported that bypassing the account via direct platform integration might also end, because "Hue plans to make a Hue account mandatory for the API in the foreseeable future" — [Born's Tech, 2023-10-05](https://borncity.com/win/2023/10/05/philips-hue-lights-require-cloud-account-from-2024/). No 2025-2026 source confirming enforcement on the local API was found (see Gaps).

#### Hue Remote API (cloud)

- Remote access means registering a "Remote Hue API" app on the Hue developer portal to get OAuth credentials, then running the OAuth 2.0 authorization-code flow against api.meethue.com. The V1 `/oauth2` endpoint was deprecated in favor of `/v2/oauth2` (announcement dated around 2020) — [Hue "New Hue API" page (search extract)](https://developers.meethue.com/new-hue-api/); [GoToGuy blog](https://gotoguy.blog/2020/05/21/remote-authentication-and-controlling-philips-hue-api-using-postman/).
- CLIP v2 can be reached through the cloud gateway at `api.meethue.com/route` — [LiveContext integration page (search extract)](https://livecontext.ai/integrations/philips-hue).
- A conflicting signal: one third-party app's App Store listing says it works only on the local network "as Philips does not allow third-party apps to access the Hue Bridge from outside the local network without contractual regulations" — [all 4 hue App Store listing (search extract)](https://apps.apple.com/app/id1092855479).

#### Libraries (release data fetched 2026-10-17)

- Python: `aiohue` 4.9.0, released 2026-07-30. HA's Hue integration pins it (`aiohue==4.9.0`, `iot_class: local_push`, zeroconf `_hue._tcp.local.`) — [PyPI aiohue](https://pypi.org/project/aiohue/); [HA hue manifest.json](https://github.com/home-assistant/core/blob/dev/homeassistant/components/hue/manifest.json). Release 4.8.1 added customizable effects through the `effects_v2` API. No release note mentions the Bridge Pro or certificates — [aiohue releases](https://github.com/home-assistant-libs/aiohue/releases).
- Node.js:
  - `node-hue-api` latest is 5.0.0-beta.16 from 2023-04-10 (stale) — [npm node-hue-api](https://www.npmjs.com/package/node-hue-api).
  - `hue-sync` (TypeScript CLIP v2) is 0.1.3 on npm (2022-08-11), though the GitHub topic page shows repository activity in March 2026 — [npm hue-sync](https://www.npmjs.com/package/hue-sync); [GitHub topic hue-api-v2](https://github.com/topics/hue-api-v2).
- Go: `cB-Sigma/hue` covers CLIP v2 resources, the event stream, link-button pairing, bridge certificate checks, and v1 for old bridges — [GitHub cB-Sigma/hue](https://github.com/cB-Sigma/hue).
- .NET: HueApi (Q42.HueApi) states it works with the Hue Bridge Pro and supports the remote API — [NuGet HueApi](https://www.nuget.org/packages/HueApi/); [Q42.HueApi README](https://github.com/michielpost/Q42.HueApi/blob/master/README.md).
- Java: the openHAB Hue binding (Java) implements API v2 with SSE — [openHAB Hue binding](https://www.openhab.org/addons/bindings/hue/). Java clients can also be generated from the OpenHue OpenAPI spec — [openhue-api](https://github.com/openhue/openhue-api).

#### Hue through Matter

- HA docs: "the communication over Matter only provides the basic controls over lights, while the official Hue integration brings all Hue unique features like (dynamic) scenes or entertainment mode" — [HA Matter integration](https://www.home-assistant.io/integrations/matter/).

### Inferences

- **A browser cannot call the bridge directly.** The Bridge Pro (and soon the square bridge) presents a Signify-CA certificate on a LAN IP, which a browser will not trust by default, and the bridge sends no CORS headers suited to a web app (not verified). So a browser dashboard on a tablet should go through a small backend (Node, Java or Python) or HA, which holds the application key, validates the certificate against Signify's CA, and relays the SSE event stream to the page, for example over WebSocket or SSE.
- **Discovery:** the backend should use mDNS (`_hue._tcp`) or a fixed IP, not UPnP, so it keeps working after a Bridge Pro migration.
- **Library choice for a Node or Java backend:** no well-maintained, Bridge-Pro-aware library was found for either language. Calling CLIP v2 directly (a small surface: GET/PUT resources plus one SSE stream), or generating a client from OpenHue, looks safer than depending on `node-hue-api`.
- **The Remote API is not needed** if a home server or HA is on the LAN. It only matters if the dashboard must work without any LAN backend.

### Gaps

- Official Signify documentation (developers.meethue.com) could not be fetched. Not verified:
  - The exact pairing call (from prior knowledge: `POST /api` with `{"devicetype":..., "generateclientkey": true}`).
  - The `hue-application-key` header name.
  - Published command rate limits (commonly cited as about 10 light commands per second and 1 group command per second).
  - Remote API quotas and app-approval rules.
- No 2025-2026 confirmation was found on whether a Hue account is now required to create local API keys or use the local API.
- The US price of the Bridge Pro, and whether Signify publishes its CA root for client validation, were not found.

---

## Q2. Common Wi-Fi brands: official APIs, auth, limits, local options, HA support, problem brands

### Takeaway

Ranked by local and API friendliness for a dashboard:

1. **Best:** Shelly (documented local HTTP/WebSocket/MQTT, HA local push) and LIFX (documented LAN protocol and a token-based cloud API).
2. **Good:** Govee (official LAN API you enable per device, plus an official cloud API with an API key) and TP-Link Kasa/Tapo (no official public API, but HA and python-kasa control them locally; newer firmware needs TP-Link cloud credentials and a "Third-Party Compatibility" toggle).
3. **Workable:** Tuya/Smart Life (official HA cloud integration via QR code; local control needs per-device keys from a Tuya developer account) and Meross (no public API; HACS `meross_lan` works locally).
4. **Avoid:** Wyze (cloud only, unofficial API, key portal, may be cut off) and Sengled (Alexa skill removed August 2025, unmaintained HA integration).

### Cited Findings

#### Govee

- Official cloud "Platform API" at developer.govee.com. You get the API key in the Govee Home app (Account > Settings > About Us > Apply for API Key), and the key is emailed — [hacs-govee README](https://github.com/LaggAt/hacs-govee); [govee2mqtt README](https://github.com/wez/govee2mqtt).
- Cloud rate limits (secondary sources, conflicting):
  - 10,000 requests per API key per rolling 24 h, with HTTP 429 when exceeded — [apis.io Govee rate limits](https://apis.io/rate-limits/govee/govee-rate-limits/); [HA community thread](https://community.home-assistant.io/t/govee-integration/228516?page=58).
  - About 10 requests per device per minute — [node-govee-led on npm](https://www.npmjs.com/package/node-govee-led).
  - "60 requests per minute" — [hacs-govee README](https://github.com/LaggAt/hacs-govee).
- The official Govee LAN API must be enabled per device in the Govee Home app. It uses UDP multicast discovery (group 239.255.255.250), and not all Govee devices support it — [govee2mqtt CONFIG.md](https://github.com/wez/govee2mqtt/blob/main/docs/CONFIG.md); [Govee WLAN guide (linked from HA)](https://app-h5.govee.com/user-manual/wlan-guide).
- HA has an official **Govee lights local** integration (Local Push) that uses the LAN API — [HA govee_light_local](https://www.home-assistant.io/integrations/govee_light_local/). Its library `govee-local-api` is at 3.1.0, released 2026-09-08 — [PyPI govee-local-api](https://pypi.org/project/govee-local-api/).
- Community `govee2mqtt` is LAN-first. It also uses Govee's "undocumented and likely unsupported" AWS IoT MQTT service (with account email and password) for fast status updates, and the Platform API as fallback — [govee2mqtt README](https://github.com/wez/govee2mqtt).
- Govee ships Matter on newer products: CES 2025 Matter lights, Strip Light 2, Outdoor Spotlights 2, and Ceiling Light Ultra H1270 (Matter 1.3) — [AppleInsider CES 2025](https://appleinsider.com/articles/25/01/05/two-new-govee-matter-smart-lights-have-arrived-at-ces-2025); [Notebookcheck strip](https://www.notebookcheck.net/Govee-launches-new-smart-LED-light-strip-with-Matter-support.1364941.0.html); [Notebookcheck ceiling](https://www.notebookcheck.net/New-Govee-Ceiling-Light-Ultra-with-Matter-on-the-way.1182810.0.html).

#### TP-Link Kasa and Tapo

- HA **TP-Link Smart Home** integration (Local Polling) covers Kasa and Tapo plugs, bulbs, switches, hubs and cameras. "If your device is a newer Kasa or Tapo device it will require your TP-Link cloud username and password to authenticate for local access." Tapo devices always require authentication. HA suggests disabling automatic firmware updates on older devices that don't yet need authentication — [HA tplink](https://www.home-assistant.io/integrations/tplink/).
- Some firmware requires turning on "Third-Party Compatibility" (Tapo app: Tapo Lab > Third-Party Compatibility; Kasa app: Settings > Third-Party Compatibility). The alternative is to factory reset and block the device's internet access — [HA tplink](https://www.home-assistant.io/integrations/tplink/).
- `python-kasa` (the library behind HA) is at 0.11.0.1, released 2026-10-04, and supports both Kasa and Tapo — [PyPI python-kasa](https://pypi.org/project/python-kasa/); [python-kasa README](https://github.com/python-kasa/python-kasa).
- The Node `tplink-smarthome-api` was last released 2023-11-15 (5.0.0) — [npm tplink-smarthome-api](https://www.npmjs.com/package/tplink-smarthome-api).
- TP-Link sells Matter models: the Tapo Mini plug, the Kasa KS205 Matter switch, and others — [Reviewed](https://amp-analytics.reviewed.com/smarthome/features/matter-smart-home-devices-to-buy); [Selora (integrator)](https://selorahomes.com/devices/tp-link/).

#### Wyze

- Wyze offers a portal for generating an API key for "Home Bridge or other third-party apps (aside from Google, Alexa, or IFTTT)", signed in with Wyze app credentials — [Wyze support article](https://support.wyze.com/hc/en-us/articles/16129834216731).
- Wyze publishes no full endpoint reference. The key pair is exchanged for an access token (about 2 days) and a refresh token (about 30 days) — [apis.io Wyze](https://apis.io/apis/wyze/developer-api/). Community reports say keys expire yearly — [docker-wyze-plug-api](https://github.com/mackenly/docker-wyze-plug-api).
- The HA option is the HACS custom `ha-wyzeapi`, which uses "the unofficial API... access may be cut off at anytime". It controls bulbs and plugs. Camera motion sensing was "Disabled following API usage incident with Wyze" — [ha-wyzeapi README](https://github.com/SecKatie/ha-wyzeapi).
- `wyze-sdk` (Python) is at 2.3.8, released 2026-08-20 — [PyPI wyze-sdk](https://pypi.org/project/wyze-sdk/). There is no official HA integration (a docs page `wyze` does not exist in HA's docs repository).

#### Meross

- No official public API was found. The community HACS integration `meross_lan` supports direct HTTP on the LAN, a local MQTT broker, and Meross cloud MQTT. Devices can stay paired with the official app, and the per-device key can be recovered through the Meross cloud account. The default polling period is 30 s — [meross_lan README](https://github.com/krahabb/meross_lan).
- The `meross-iot` Python library is at 0.4.10.4, released 2026-01-22 — [PyPI meross-iot](https://pypi.org/project/meross-iot/). There is no official HA Meross integration (no `meross` page in the HA docs repository).

#### LIFX

- HA **LIFX** integration (Local Polling) controls lights "over your local network, without going through the LIFX cloud" and polls each light every 10 s. HomeKit Controller is an alternative with push updates and encryption. The LIFX Switch needs Matter (firmware 4.100+) or HomeKit — [HA lifx](https://www.home-assistant.io/integrations/lifx/); [LIFX Matter firmware article](https://support.lifx.com/hc/en-us/articles/34857864915095-Matter-Firmware-Updates-for-LIFX-Devices).
- The LIFX cloud HTTP API uses a personal access token from cloud.lifx.com/settings. A third-party package reports a limit of 60 requests per 60 s per token with HTTP 429 when exceeded, which the official page could not confirm — [node-red-contrib-lifx-api](https://flows.nodered.org/node/node-red-contrib-lifx-api); [LIFX rate-limits page (not fetched)](https://api.developer.lifx.com/reference/rate-limits).
- `aiolifx` (LAN) is at 1.2.2, released 2026-05-26 — [PyPI aiolifx](https://pypi.org/project/aiolifx/).

#### Tuya / Smart Life (white-label)

- The official HA **Tuya** integration (Cloud Push) now signs in with the Smart Life or Tuya Smart app: you enter a "User Code" and scan a QR code. It uses Tuya's official `tuya-device-sharing-sdk`, which "does not expose all functionality available in SmartLife" — [HA tuya](https://www.home-assistant.io/integrations/tuya/).
- Local LAN control (protocols 3.1-3.5) needs each device's `local_key`. TinyTuya's wizard gets it from a Tuya IoT developer account (iot.tuya.com). "The IoT Core subscription is a free trial that expires after one month" and must be renewed (extendable for 1, 3 or 6 months) to fetch keys for new devices — [TinyTuya README](https://github.com/jasonacox/tinytuya).
- TinyTuya is at 1.20.0, released 2026-07-06 — [PyPI tinytuya](https://pypi.org/project/tinytuya/). The local HA options are the custom integrations LocalTuya (xZetsubou fork) and `tuya-local` — [hass-localtuya](https://github.com/xZetsubou/hass-localtuya); [tuya-local](https://github.com/make-all/tuya-local).

#### Sengled (problem brand)

- Around June 18, 2025, Sengled Wi-Fi bulbs stopped responding across Alexa, Google Home and the Sengled app. From August 1, 2025, Amazon removed Sengled's Alexa skill after "a series of prolonged outages". Zigbee, Matter or Bluetooth Sengled bulbs may still work with Alexa directly, but Wi-Fi-only bulbs that relied on the skill lost their Alexa path — [PCWorld](https://pcworld.com/article/2864586/amazon-yanks-sengleds-alexa-skill-after-a-series-of-service-outages.html); [inside.lighting](https://inside.lighting/news/25-06/smart-lighting-cloud-failure-leaves-users-dark); [LightNOW](https://www.lightnowblog.com/2025/08/sengled-smart-lamps-service-outages-alexa-skill-shut-down).
- An April 2026 status-tracker snapshot reports the app working but "will no longer link to Alexa". This is a low-quality source — [UpDownRadar](https://updownradar.com/status/Sengled.com).
- The only HA option, the HACS `ha-sengledapi` (unofficial API), says: "I no longer plan on supporting this integration" — [ha-sengledapi README](https://github.com/jfarmer08/ha-sengledapi).

#### Shelly

- The HA **Shelly** integration (Local Push) supports all four device generations. Gen1 uses CoIoT, and Gen2+ uses RPC with push and no extra setup ("cloud connection is not needed"). Firmware 2.0.0 "Enhanced Security" turns on HTTPS. Shelly BLU sensors go through BTHome — [HA shelly](https://www.home-assistant.io/integrations/shelly/).
- `aioshelly` is at 13.35.0, released 2026-10-15 — [PyPI aioshelly](https://pypi.org/project/aioshelly/).
- The Shelly Cloud Control API is authenticated with an auth key from the app, which is invalidated when the account password changes — [Shelly KB article](https://www.shelly.com/de/blogs/documentation/kbuca-understanding-the-differences-between-shelly). Its rate limit is about 1 request per second per account (third-party client README, unconfirmed) — [hex.pm shelly](https://readme.hex.pm/shelly/0.3.0). The official API docs are at shelly-api-docs.shelly.cloud (not fetched).
- Brief note on SwitchBot (sensors, see Q9): the cloud API allows 10,000 requests per day — [HA switchbot_cloud](https://www.home-assistant.io/integrations/switchbot_cloud/).

### Inferences

- For a self-built backend, the lowest-effort, lowest-risk path is to let HA own all the Wi-Fi brands and have the dashboard talk only to HA (REST plus WebSocket). Re-implementing TP-Link authentication, Tuya local keys, Govee multicast or Meross signing in Node or Java would be costly.
- Wyze and Sengled devices are poor long-term candidates. Replacing them with Matter or Shelly devices is probably cheaper than integrating them.

### Gaps

- Govee's official rate limits and LAN-capable model list, LIFX's official rate limits, and Shelly's cloud limits were not verified on the vendors' own pages (they could not be fetched).
- No official public developer API was found for TP-Link (Kasa or Tapo) or Meross. Absence was not confirmed on the vendors' sites.
- No 2026 status for Sengled's cloud from Sengled itself was found.

---

## Q3. Matter and Thread in 2026: local control with multi-admin alongside Alexa; migrating Alexa-only Wi-Fi devices

### Takeaway

Yes. Home Assistant's Matter integration controls Matter Wi-Fi and Thread devices locally. Each device supports at least 5 fabrics, so the same bulb or plug can stay in Alexa and also be shared to HA, using an 11-digit pairing code from the Alexa app. Migration only works for devices whose hardware or firmware supports Matter. A non-Matter Wi-Fi bulb set up through a brand skill in Alexa cannot be converted; it needs its native integration or replacement.

### Cited Findings

- HA Matter integration: "Matter products run locally and always allow local control... without the need for any internet connection or cloud services." HA runs its own Matter controller as a separate "Matter Server" app (add-on) — [HA matter](https://www.home-assistant.io/integrations/matter/).
- Multi-fabric: "you can join the same device to multiple controllers... The standard describes that each device should be able to at least support 5 different fabrics simultaneously." Devices can be shared from Google or Apple into HA, and from HA to other platforms ("Manage fabrics" lists, for example, Google LLC) — [HA matter](https://www.home-assistant.io/integrations/matter/).
- Sharing from Alexa (TP-Link's instructions): in the Alexa app open the device's Settings, tap "Other Assistants and Apps", then "Add Another" to create an 11-digit setup code. The code expires in 15 minutes, a fresh code is needed for each additional ecosystem, and the phone and device must be on the same 2.4 GHz network — [TP-Link FAQ 3573](https://www.tp-link.com/nl/support/faq/3573); [Omada US FAQ 3573](https://www.omadanetworks.com/us/support/faq/3573/).
- Thread devices need a Thread border router. Options include Nest Hub (2nd gen), Nest Wifi Pro, HomePod mini or Apple TV 4K, or HA's OpenThread Border Router app with Connect ZBT-1/ZBT-2/SkyConnect. A Thread logo does not imply Matter: look for the Matter logo — [HA matter](https://www.home-assistant.io/integrations/matter/).
- Wi-Fi Matter devices "often use the 2.4 GHz frequency". Commissioning uses the HA Companion app on the phone (Bluetooth for pairing only) — [HA matter](https://www.home-assistant.io/integrations/matter/).
- Supported installs: the Matter Server app on HA OS is the only supported option. Container users can run the `matter-js/matterjs-server` Docker image — [HA matter](https://www.home-assistant.io/integrations/matter/).
- The Python Matter Server 8.1.2 (2025-12-15) "is the final version". It was rewritten as the matter.js-based server, a drop-in replacement that supports Matter 1.6.0 — [python-matter-server README](https://github.com/home-assistant-libs/python-matter-server); [PyPI python-matter-server](https://pypi.org/project/python-matter-server/). The matterjs-server README still labels itself "Beta... not yet officially re-certified by the CSA" — [matterjs-server README](https://github.com/matter-js/matterjs-server).
- For a self-built controller in JavaScript, `@matter/main` (matter.js) is at 0.17.9, released 2026-08-06 — [npm @matter/main](https://www.npmjs.com/package/@matter/main).
- Matter can be less capable than native integrations. Hue over Matter gives only basic light control — [HA matter](https://www.home-assistant.io/integrations/matter/).
- Some vendors add Matter by firmware update, for example the LIFX Switch on firmware 4.100+ — [HA lifx](https://www.home-assistant.io/integrations/lifx/); [LIFX Matter firmware](https://support.lifx.com/hc/en-us/articles/34857864915095-Matter-Firmware-Updates-for-LIFX-Devices).
- Some vendors require an account before Matter can be enabled on certain products — [HA matter](https://www.home-assistant.io/integrations/matter/).

### Inferences

- Practical migration:
  1. Identify each Alexa device (Q4).
  2. If it is a Matter device (Matter logo or QR code, or "Other Assistants and Apps" shows a pairing option), share it from Alexa to HA. Alexa keeps working.
  3. If it is a brand-cloud device (Kasa, Govee, Tuya and so on), add it to HA through the native integration instead.
  4. If it is Wyze or Sengled Wi-Fi, plan to replace it, preferably with Matter devices, which then join both Alexa and HA.
- A self-built (non-HA) backend could use matter.js directly. But that means owning commissioning, fabric credentials and OTA quirks. HA's Matter Server is the lower-risk choice.

### Gaps

- Amazon's own help page for "Other Assistants and Apps" could not be retrieved. The menu names come from TP-Link FAQs.
- No authoritative per-brand list of 2026 Matter models (TP-Link, Meross, Govee) was found.

---

## Q4. How to find out which brand and protocol an Alexa-set-up device is

### Takeaway

No authoritative Amazon guide was found. The practical approach combines four checks:

- The Alexa app: which skill the device came from, and whether its settings offer "Other Assistants and Apps", which indicates a Matter device.
- The physical label or packaging: brand, a Matter QR code or logo.
- The router's DHCP client list, where the MAC vendor prefix identifies the maker.
- Home Assistant's automatic discovery (zeroconf, DHCP, Bluetooth), which flags many brands once HA is on the network.

### Cited Findings

- Brand-cloud devices reach Alexa through the brand's skill ("the skill that corresponds to your smart bulb or switch brand"). A forum fix of disabling and re-enabling the Wyze skill shows devices are tied to the skill — [support.com TP-Link guide](https://www.techsolutions.support.com/how-to/how-to-add-tp-link-light-bulbs-to-alexa-12445); [Wyze forum](https://forums.wyze.com/t/wyze-bulb-alexa/42983).
- Some bulbs pair directly in the Alexa app with no maker app, for example a Sengled bulb set up from a code printed on the bulb (anecdotal) — [Reddit r/alexa thread](https://lr.in.psf.lt/r/alexa/comments/1glcblx/smart_lights_alexa_routine).
- For Matter devices, the Alexa device Settings page shows "Other Assistants and Apps", which generates a Matter pairing code — [TP-Link FAQ 3573](https://www.tp-link.com/nl/support/faq/3573).
- HA discovers devices automatically through DHCP MAC prefixes and zeroconf. For example, the Nest integration matches MAC prefixes `18B430`, `641666`, `D8EB46`, and the Hue integration matches zeroconf `_hue._tcp.local.` — [HA nest manifest](https://github.com/home-assistant/core/blob/dev/homeassistant/components/nest/manifest.json); [HA hue manifest](https://github.com/home-assistant/core/blob/dev/homeassistant/components/hue/manifest.json).
- Thread and Matter are separate logos, so check for the Matter logo specifically — [HA matter](https://www.home-assistant.io/integrations/matter/).

### Inferences

- A one-evening checklist for the owner:
  1. In the Alexa app, open More > Skills & Games > Your Skills and list the linked brand skills.
  2. For each light, open Devices > device > Settings and note any skill name, manufacturer text, and whether "Other Assistants and Apps" exists (which means Matter).
  3. Read the label on the bulb or plug.
  4. Match the router's client list by MAC OUI.
  5. Start HA and accept its discovered-device prompts.
- Devices that pair only with Alexa, show no maker skill and have no Matter option are probably Alexa-local (Zigbee on an Echo hub, or Amazon "Frustration-Free Setup" devices). These may not be reachable by any third-party API without re-pairing. This is unverified; see Gaps.

### Gaps

- No Amazon documentation was found describing exactly how the Alexa app labels a device's source skill or manufacturer, or how Amazon Basics bulbs and plugs can be accessed by third parties.

---

## Q5. Google Nest Device Access / Smart Device Management (SDM) API

### Takeaway

SDM is the official cloud API for all Nest thermostat models still supported. It costs a one-time, non-refundable $5 and needs a consumer Google account, a Google Cloud project (SDM API plus Pub/Sub API), an OAuth client, and a Pub/Sub topic for push events.

It gives the owner:

- Ambient temperature and humidity.
- Mode (HEAT, COOL, HEATCOOL, OFF).
- Heat and cool setpoints.
- Eco mode and eco setpoints.
- HVAC status.
- Fan timer and connectivity.

Personal (sandbox) projects allow up to 25 users across 5 structures. The OAuth app must be published to "In production" status, or refresh tokens stop working every 7 days.

### Cited Findings

- Fee and accounts: SDM "requires a US$5 fee" (non-refundable). Google Workspace accounts are not supported (consumer accounts only). The Google account tied to a Device Access project cannot be changed. Advanced Protection Program accounts are blocked from the required "Restricted" scopes — [HA nest](https://www.home-assistant.io/integrations/nest/).
- Setup steps:
  1. Create a Google Cloud project and enable the Smart Device Management API and the Cloud Pub/Sub API.
  2. Configure the OAuth consent screen as External, add yourself as a test user, then "Publish app" to "In production". "Make sure the status is not Testing, or you will get logged out every 7 days."
  3. Create a Web application OAuth client.
  4. Create the Device Access project ($5) with that client ID.
  5. Create a Pub/Sub topic and grant `sdm-publisher@googlegroups.com` the Pub/Sub Publisher role on the topic, then enable events with that topic in the Device Access Console.

  The Pub/Sub process "changed as of January 23rd 2025" — [HA nest](https://www.home-assistant.io/integrations/nest/).
- Leaving the consent screen's logo or domain fields empty avoids Google's verification process. Users will see a "Google hasn't verified this app" screen and click through it — [HA nest](https://www.home-assistant.io/integrations/nest/).
- Sandbox limits: "A developer account in the Sandbox is limited to 25 users across 5 structures across all projects", 5 users per structure (the owner doesn't count), and up to 3 projects per account — [Device Access User and Rate Limits (search extract)](https://developers.google.com/nest/device-access/project/limits).
- Rate limits (from localized copies of the same page; re-check the English page) — [Device Access limits (search extract, ja/it pages)](https://developers.google.com/nest/device-access/project/limits?hl=ja):
  - `devices.executeCommand` is limited to 10 queries per minute (QPM) per project per user, and `devices.list` to 5 QPM.
  - Each trait command is limited to 5 QPM per project, user and device.
  - Device-instance limits for a THERMOSTAT are 5 QPM or 100 queries per hour (QPH), shared across all projects controlling that device.
  - Exceeding a limit returns RESOURCE_EXHAUSTED.
- Supported devices: "All Google Nest thermostat models are supported in the Device Access program and the API functionality is the same for all models." Separate Nest temperature sensors are not included — [Supported Devices (search extract)](https://developers.google.com/nest/device-access/supported-devices); [Thermostat API guide (search extract)](https://developers.google.com/nest/device-access/api/thermostat).
- HA notes that only the currently "Active Sensor" temperature is reported, so extra Nest sensors are not exposed — [HA nest](https://www.home-assistant.io/integrations/nest/).
- Thermostat traits, as implemented by the reference client used by HA: `Info`, `Connectivity`, `Fan`, `Humidity`, `Temperature`, `ThermostatEco`, `ThermostatHvac`, `ThermostatMode`, `ThermostatTemperatureSetpoint` — [python-google-nest-sdm traits.py](https://github.com/allenporter/python-google-nest-sdm/blob/main/google_nest_sdm/traits.py). Fields include:
  - ThermostatEco: `availableModes`, `mode` (default "OFF"), `heatCelsius`, `coolCelsius`.
  - ThermostatHvac: `status`.
  - ThermostatMode: `availableModes`, `mode`.
  - ThermostatTemperatureSetpoint: `heatCelsius`, `coolCelsius`, with commands SetHeat, SetCool and SetRange.

  Source: [thermostat_traits.py](https://github.com/allenporter/python-google-nest-sdm/blob/main/google_nest_sdm/thermostat_traits.py). Google's trait docs say ThermostatHvac reports values such as HEATING — [Traits (search extract)](https://developers.google.com/nest/device-access/traits).
- Google publishes a codelab, "Building a Device Access Web Application", so SDM can be called from a web app with OAuth — [Device Access web-app codelab (search extract)](https://developers.google.com/nest/device-access/codelabs/web-app).
- The API overview advises that devices authorized through SDM should not be synced to Home Graph for Google Assistant integrations, to avoid conflicts — [SDM API overview (search extract)](https://developers.google.com/nest/device-access/api).

### Inferences

- **A small backend is the clean design.** SDM events arrive through a Google Cloud Pub/Sub subscription, and the OAuth refresh token must be stored safely. So a backend or HA should hold the token, subscribe to Pub/Sub, and push changes to the tablet page. Putting a refresh token in the wall tablet's browser is a security risk.
- **Rate limits are not a problem** for one home: 5 commands per minute per thermostat is plenty for a dashboard. Read state from Pub/Sub events or occasional GETs rather than tight polling.
- **Costs:** besides the $5, a single home's Pub/Sub traffic is expected to stay within the free tier, but Google Cloud billing terms were not checked.

### Gaps

- Official Google pages could not be fetched. The 5 QPM / 100 QPH thermostat figure comes from a localized page.
- Whether SDM exposes temperature scale (a Settings trait) or `HEAT`/`COOL`/`HEATCOOL`/`OFF` naming exactly was not re-verified on Google's site (HEAT/COOL/HEATCOOL/OFF is from prior knowledge).
- Pub/Sub cost and quota for this use were not checked.

---

## Q6. 2025-2026 changes: end of support for Nest Learning 1st/2nd gen, 4th-gen Learning Thermostat, Google Home APIs

### Takeaway

- **1st and 2nd gen Learning Thermostats:** lost all app, remote and third-party connectivity on October 25, 2025. They now run only local schedules. If the owner has one, no API (SDM or other) can reach it.
- **4th gen (2024):** supports Matter and SDM.
- **Google Home APIs:** native Android and iOS SDKs (public beta through at least February 2026, certification required). They do not help a browser dashboard unless the owner writes a native Android or iOS app.

### Cited Findings

- End of support: "starting on October 25, 2025" the Nest Learning Thermostat 1st gen (2011), 2nd gen (2012) and 2nd gen Europe (2014) "will no longer connect to or work in the Google Nest app or Google Home app". Remote control, notifications, third-party assistant features, Home/Away Assist and Nest Protect links all end. Pre-set schedules "will continue to work". A discounted 4th-gen offer ran until December 31, 2025 — [Google support: End of support for Nest Learning Thermostats (1st & 2nd gen)](https://support.google.com/googlenest/answer/16233096) (search extract); [9to5Google 2025-04-25](https://9to5google.com/2025/04/25/nest-learning-thermostats-1st-2nd-gen/).
- 4th-gen Learning Thermostat:
  - Google's spec page: "Control the Nest Learning Thermostat with Google Home and other Matter-enabled apps and voice assistants". Setup requires the Google Home app. Wi-Fi is 802.11n at 2.4/5 GHz. The bundle includes the Nest Temperature Sensor (2nd gen) — [Google Store specs (search extract)](https://store.google.com/us/product/nest_learning_thermostat_4th_gen_specs).
  - Reviewers confirm Apple Home works via Matter after Google Home setup — [AppleInsider review, 2024-08-27](https://appleinsider.com/articles/24/08/27/nest-learning-thermostat-4th-gen-review-finally-works-for-apple-home-users); [BGR review](https://bgr.com/reviews/nest-learning-thermostat-4th-generation-review-theres-no-better-smart-thermostat/).
- Google Home APIs:
  - Announced May 2024 for Android and iOS, covering devices in Google Home "such as Matter light bulbs or the Nest Learning Thermostat" — [Google Developers Blog](https://developers.googleblog.com/en/home-apis-enabling-all-developers-to-build-for-the-home/).
  - The Android public developer beta opened in January 2025 (up to 100 users during beta), with an iOS beta to follow and general availability "anticipated" by end of 2025 — [SD Times](https://sdtimes.com/google/google-home-apis-now-available-as-public-developer-beta-on-android/); [9to5Google 2025-01-07](https://9to5google.com/2025/01/07/google-home-apis-dev-beta/).
  - The iOS release notes list "Home APIs Public Beta release 1.7" dated 2026-02-09 (iOS 17+) — [Home APIs iOS release notes (search extract)](https://developers.home.google.com/apis/ios/release-notes).
  - Apps must pass certification before launch — [Google Developers Blog](https://developers.googleblog.com/en/home-apis-enabling-all-developers-to-build-for-the-home/).

### Inferences

- The owner should first check the thermostat's generation, in the Google Home app under device settings, or on the base. A 1st or 2nd gen means replacement (for example a 4th gen) is needed before any dashboard integration.
- The Home APIs could matter only if the wall tablet runs Android and the owner is willing to build a native app or wrapper. For a web page, SDM (cloud) or Matter (local, via HA) are the realistic routes.

### Gaps

- Whether the Home APIs reached general availability, or gained a web or REST surface, by October 2026 was not found.
- No source confirms whether the 4th gen exposes Matter over Thread as well as Wi-Fi. An FCC-era report suggested no Thread — [Notebookcheck](https://www.notebookcheck.net/Google-s-Nest-Learning-Thermostat-gets-a-borderless-makeover.868815.0.html). A Matter Alpha listing mentions "Thread beta" — [Matter Alpha](https://www.matteralpha.com/product/nest-learning-thermostat-4th-gen-1897/features).

---

## Q7. Matter route for Nest thermostats

### Takeaway

Only the Nest Thermostat (2020) and the Nest Learning Thermostat (4th gen) expose Matter. The 3rd-gen Learning Thermostat and the Thermostat E do not. Sharing the thermostat from Google Home to HA's Matter fabric gives local control of:

- Current temperature.
- Heat and cool setpoints.
- System mode.
- Possibly running state (hvac_action).

Reportedly missing: fan control, eco and comfort presets, and the remote temperature sensors. It avoids the $5 fee, the Cloud project and Pub/Sub.

### Cited Findings

- Models (secondary guides):
  - Nest Thermostat (2020) "Supports Matter via firmware update (version 1.3 or later)". Dates conflict: "late 2024" versus an April update — [Leios guide](https://www.leios.consulting/guides/nest-thermostat-home-assistant/); [TechRadar](https://www.techradar.com/news/nest-thermostat-update-could-finally-kickstart-the-matter-smart-home-era).
  - The Learning Thermostat 3rd gen and Nest Thermostat E do not support Matter — [Leios guide](https://www.leios.consulting/guides/nest-thermostat-home-assistant/).
  - The 2020 Nest Thermostat was "the first-ever Matter-certified thermostat" — [Matter Alpha explainer](https://www.matteralpha.com/explainer/best-matter-compatible-home-thermostats).
- Setup: in Google Home, use "Linked Matter apps and services" to get a code, then add it in HA under Settings > Devices & Services > Matter — [joinhomeshift guide](https://joinhomeshift.com/blog/nest-thermostat-home-assistant). HA's generic flow for sharing from Google Home is documented in [HA matter](https://www.home-assistant.io/integrations/matter/).
- What Nest exposes over Matter, from user reports:
  - Setpoint and on/off/heat/cool modes work, but fan control was not available — [HA community: Nest thermostat + matter](https://community.home-assistant.io/t/nest-thermostat-matter-local-hvac-control/603058).
  - "only a climate entity is exposed". The additional temperature sensors and the presets (comfort, eco, sleep) are not exposed — [HA community: Nest Gen 4](https://community.home-assistant.io/t/nest-gen-4/791667).
  - Room sensors don't support Matter — [AppleInsider](https://appleinsider.com/articles/24/08/27/nest-learning-thermostat-4th-gen-review-finally-works-for-apple-home-users).
- HA's Matter climate entity reads these Matter Thermostat-cluster attributes when the device provides them: `LocalTemperature`, `OccupiedHeatingSetpoint`, `OccupiedCoolingSetpoint`, `SystemMode`, `ControlSequenceOfOperation`, `ThermostatRunningState` (mapped to `hvac_action` "if it exists"), `ThermostatRunningMode`, setpoint limits, `Occupancy`, and the Matter 1.4 preset attributes `Presets`, `ActivePresetHandle` and `PresetTypes` — [HA core matter/climate.py](https://github.com/home-assistant/core/blob/dev/homeassistant/components/matter/climate.py).

### Inferences

- **Best combination:** use Matter (local, low latency) for the dashboard's temperature and setpoint tiles. Add SDM only if humidity, eco state or HVAC status are wanted and Matter does not provide them. Both can run at once, since multi-admin keeps Google Home as the primary controller.
- Matter needs HA (or another Matter controller such as matter.js) on the LAN. That fits the "small home server or HA" architecture.

### Gaps

- Whether Nest over Matter exposes `ThermostatRunningState` (hvac_action) or a humidity cluster was not confirmed. A forum question about hvac_action went unanswered — [HA community](https://community.home-assistant.io/t/nest-thermostat-matter-local-hvac-control/603058).
- No official Google document listing the Matter clusters that Nest thermostats expose was found.

---

## Q8. Home Assistant Nest integration: setup complexity, reliability, "Works with" status

### Takeaway

The HA Nest integration is official, actively maintained, and push-based (Pub/Sub). Its setup is the most error-prone part, with roughly 10 console steps across three Google consoles. Once set up it is reliable, provided the OAuth app is in production status and a consumer Google account is used.

### Cited Findings

- `iot_class: cloud_push`. Codeowner `@allenporter`. Requires `google-nest-sdm==9.2.1` (released 2026-08-16) — [HA nest manifest.json](https://github.com/home-assistant/core/blob/dev/homeassistant/components/nest/manifest.json); [PyPI google-nest-sdm](https://pypi.org/project/google-nest-sdm/).
- Entities: thermostats appear as `climate` entities. Sensors cover Temperature, Humidity and a Fan timer timestamp. State changes arrive through the Cloud Pub/Sub subscriber — [HA nest](https://www.home-assistant.io/integrations/nest/).
- The docs list common setup failures and fixes: "Can't link to [Project Name]" (OAuth client ID mismatch), "Error 400: redirect_uri_mismatch", "Error 403: access_denied" (consent screen triggering verification), and "No access to partner information" (account cannot access the Home, or a legacy Nest account not migrated) — [HA nest troubleshooting](https://www.home-assistant.io/integrations/nest/).
- HA uses My Home Assistant (`https://my.home-assistant.io/redirect/oauth`) as the OAuth redirect, to meet Google's HTTPS and public-domain redirect rules. A self-built web app must likewise use a public HTTPS redirect URI — [HA nest](https://www.home-assistant.io/integrations/nest/).
- One Matter-versus-SDM guide notes that Matter avoids the SDM registration fee — [Leios guide](https://www.leios.consulting/guides/nest-thermostat-home-assistant/).

### Inferences

- For a technical owner, the Google Cloud setup is tedious rather than hard. The two traps are leaving the app in Testing status (re-authentication every 7 days) and using a Workspace account. If the owner builds a custom backend instead of HA, the same steps apply. They can reuse HA's documented flow as a checklist and `google-nest-sdm` (Python) as a reference implementation.

### Gaps

- "Works with Google Home" certification status of HA, or of HA's Google Assistant integration that exposes HA devices to Google Home, was not researched. It is out of scope here and covered by the hub-architecture researcher.
- No quantitative reliability data (outage frequency, token revocations) for SDM in 2025-2026 was found.

---

## Q9. Cheap extra temperature and humidity sensors with local APIs

### Takeaway

For room-by-room readings on the dashboard, the cheapest local options are Bluetooth thermometers decoded passively by HA:

- Govee H50xx/H51xx.
- Xiaomi LYWSD03MMC.
- SwitchBot Meter (about $14).
- Shelly BLU H&T (via BTHome).

Two Wi-Fi or Matter options need no Bluetooth: the Shelly H&T Gen3 (Wi-Fi plus Matter, about €41) and Aqara Zigbee sensors (need a Zigbee coordinator, and can be finicky).

### Cited Findings

- **Govee BLE** (HA `govee_ble`, Local Push): supports H5051, H5052, H5071, H5072, H5074, H5075 (active scans required), H5100, H5101 and others — [HA govee_ble](https://www.home-assistant.io/integrations/govee_ble/).
- **Xiaomi BLE** (HA `xiaomi_ble`, Local Push): listens to broadcasts "without waking the device up for polling". Devices such as the LYWSD03MMC can get bindkeys through pvvx's Telink flasher — [HA xiaomi_ble](https://www.home-assistant.io/integrations/xiaomi_ble/).
- **SwitchBot:**
  - HA `switchbot` (Local Push) works over Bluetooth and "A SwitchBot Hub is not required" — [HA switchbot](https://www.home-assistant.io/integrations/switchbot/).
  - The cloud alternative `switchbot_cloud` polls every 10 minutes. The SwitchBot Cloud API "limits users to 10,000 requests per day", and its token and secret come from the app's Developer Options — [HA switchbot_cloud](https://www.home-assistant.io/integrations/switchbot_cloud/).
  - The SwitchBot Meter is listed at about $13.99 — [SmartHomeCompared](https://smarthomecompared.com/temperature-humidity-sensors/shelly-ht-gen3-vs-switchbot-meter).
- **Shelly H&T Gen3:**
  - Direct 2.4 GHz Wi-Fi with no hub, Matter, an e-paper display, USB-C or AA power (up to about 1 year on batteries), about €40.99. Some owners report dropouts — [SmartHomeCompared](https://smarthomecompared.com/temperature-humidity-sensors/shelly-ht-gen3-vs-switchbot-meter).
  - Works with HA's native Shelly integration (local push). Battery devices may need outbound WebSocket configuration — [HA shelly](https://www.home-assistant.io/integrations/shelly/).
- **Shelly BLU H&T** goes through HA's BTHome integration (Local Push) — [HA shelly](https://www.home-assistant.io/integrations/shelly/); [HA bthome](https://www.home-assistant.io/integrations/bthome/).
- **Aqara (Zigbee)** sensors work through HA's ZHA integration, with caveats: "Xiaomi/Aqara devices... may not be fully compliant with Zigbee standards" and may need many pairing attempts. They are known not to work with some Zigbee routers (Centralite, GE, Ledvance/OSRAM, Sylvania and others) — [HA zha](https://www.home-assistant.io/integrations/zha/).

### Inferences

- If HA runs on a mini PC or Raspberry Pi with Bluetooth, or uses ESPHome Bluetooth proxies, the BLE thermometers ($5-15 each) are the cheapest route and fully local.
- If the owner wants no HA, the Shelly H&T Gen3 is the easiest to read directly, through the documented Gen2+ RPC/WebSocket API or Matter.
- Govee BLE thermometers are separate from Govee's Wi-Fi light APIs. Their readings are passive broadcasts, so they don't count against cloud rate limits.

### Gaps

- Current US prices for Govee H5075, Xiaomi LYWSD03MMC and Aqara sensors were not retrieved.
- No HA integration doc for Aqara Matter-over-Thread climate sensors (for example the W100) was checked.
- Govee H5075 and similar Govee BLE models also report to Govee's cloud through Govee gateways. Whether Govee's cloud API exposes their readings was not checked.
