---
status: concluded
date: 2026-10-17
authors: [Claude Code]
related: [2026-10-17-tablet-home-dashboard-options]
---

# Alexa/Echo and Sonos integration for a self-built tablet home dashboard (US, as of 2026-10-17)

Research method note for the report writer: `home-assistant.io`, `developer.amazon.com`, `docs.sonos.com`, `support.sonos.com` and the Sonos community forum could not be fetched directly from this research environment (blocked at the network proxy). For Home Assistant (HA) pages I read the docs' source files on the `current` branch of `github.com/home-assistant/home-assistant.io`. These are the files that build the live pages, so both URLs are given. For Amazon and Sonos pages I relied on search-engine extracts of the official pages, and I mark where a claim rests only on an extract. Library release dates come from PyPI and npm registry metadata, fetched on 2026-10-17.

## Q1. Official Amazon APIs: can a third-party app read or control Echo devices or Alexa-linked smart-home devices?

### Takeaway

No. Every official Amazon developer surface works in one of two directions. Either it exposes your device or service to Alexa (Smart Home Skill API, renamed "smart home add-ons"; Alexa Connect Kit; Matter), or it builds Alexa into your own hardware (Alexa Voice Service). Skill APIs such as Reminders and Timers act only inside a skill the user enabled, on objects that skill created. No official API lets an outside dashboard play or pause an Echo, set its volume, run Alexa routines, read Alexa's lists, or control lights and plugs that are linked to Alexa.

### Cited Findings

- The Smart Home Skill API is a "your cloud receives directives from Alexa" API. Alexa sends a `Discover` directive and your skill answers with `Discover.Response` describing devices and their capability interfaces (e.g. `Alexa.PowerController`, `Alexa.BrightnessController`). On a voice command, Alexa sends a directive to your Lambda, which calls your device cloud. Account linking is OAuth from Alexa to your authorization server, and proactive state changes go to the Alexa event gateway. — [Amazon: Connect Your Device Cloud with Smart Home Add-ons](https://developer.amazon.com/en-US/docs/alexa/smarthome/understand-the-smart-home-skill-api.html); [Amazon: Implement Smart Home Skill Code](https://developer.amazon.com/en-US/docs/alexa/smarthome/implement-your-skill.html)
- Amazon has renamed smart home skills "smart home add-ons": "No action is needed, and your existing integration will continue to operate normally." An Alexa+ variant of the same doc exists under `/docs/alexaplus/`. — [Amazon: Connect Your Device Cloud with Smart Home Add-ons](https://developer.amazon.com/en-US/docs/alexa/smarthome/understand-the-smart-home-skill-api.html); [Amazon (alexaplus docs path)](https://developer.amazon.com/ja/docs/alexaplus/smarthome/connect-your-cloud-with-addons.html)
- Amazon's smart-home development-options page lists the ways to make a device work with Alexa: a cloud skill or add-on, Matter, and Alexa Connect Kit. All of them face toward Alexa. (Seen as a search result title and URL only; contents not fetched.) — [Amazon: Smart Home Development Options](https://developer.amazon.com/en-US/docs/alexa/smarthome/development-options.html)
- Reminders API: a skill can create and manage reminders only with a customer-granted permission scope. Creating or updating needs a skill session; out-of-session tokens can only edit or delete. "A skill can only access reminders that it created itself." — [Amazon: Alexa Reminders Overview](https://developer.amazon.com/en-US/docs/alexa/smapi/alexa-reminders-overview.html); [Reminders REST API Reference](https://developer.amazon.com/en-US/docs/alexa/smapi/alexa-reminders-api-reference.html)
- Timers API: skills can create, pause, resume and cancel timers, but creating, pausing and resuming only from a skill session. Limits are 25 timers per skill and a maximum length of 2 hours. (Search extract; the pages may be 1–2 years old.) — [Amazon: Understand Alexa Timers](https://developer.amazon.com/en-US/docs/alexa/smapi/alexa-timers-overview.html); [Timers REST API Reference](https://developer.amazon.com/en-US/docs/alexa/smapi/alexa-timers-api-reference.html)
- The Alexa+ developer surfaces announced in Feb 2025 are the Alexa AI Action SDK, the AI Web Action SDK and the AI Multi-Agent SDK. They let businesses (OpenTable, GrubHub, Yelp, Tripadvisor and others) plug services or agents into Alexa+. They do not offer control of a user's Echo from outside. — [Amazon developer blog, Feb 2025: Introducing AI-native SDKs for Alexa+](https://developer.amazon.com/en-US/blogs/alexa/alexa-skills-kit/2025/02/new-alexa-announce-blog)
- A secondary source says the Action SDK is generally available and the Web Action SDK is in early access. I found no official 2026 status update. — [byteiota (secondary)](https://byteiota.com/amazon-moonraker-alexa-agentic-developer-guide/)
- The HA docs for the manual Alexa Smart Home skill confirm the direction: Alexa calls an AWS Lambda, which forwards to your HA instance, and the instance must be reachable from the internet on HTTPS port 443 with a publicly trusted certificate. — [HA: Amazon Alexa Smart Home Skill](https://www.home-assistant.io/integrations/alexa.smart_home/) ([source](https://github.com/home-assistant/home-assistant.io/blob/current/source/_integrations/alexa.smart_home.markdown))
- The unofficial projects themselves describe their method as mimicking the Alexa app: "control of Amazon Alexa devices ... using the unofficial Alexa API. Please note this mimics the Alexa app but Amazon may cut off access at anytime." — [alandtse/alexa_media_player README](https://github.com/alandtse/alexa_media_player)

### Inferences

- A dashboard that must play or pause Echos, announce, show Alexa timers and alarms, or run routines has to use an unofficial, reverse-engineered Alexa-app API (Q3) or route around Alexa entirely (Q4). There is no supported "Alexa remote control" API for consumers or hobbyists.
- A custom skill could add a voice path ("Alexa, ask Dashboard to…") that calls the owner's server. That is a voice-to-dashboard direction and does not give dashboard-to-Echo control.
- Alexa Voice Service (AVS) puts an Alexa client inside your own product. It does not reach existing Echo devices. This is from my background knowledge; the AVS page was not fetched.

### Gaps

- I could not fetch the Alexa Connect Kit or AVS pages to quote their current scope or whether either was deprecated in 2025–2026. The search returned nothing on them.
- I did not verify whether Amazon still supports the Proactive Events / Notifications API for skills in 2026. The HA manual-skill docs still rely on "Proactive Events" for change reports, which suggests the event gateway works.

## Q2. Status of Alexa list APIs (shopping / to-do)

### Takeaway

Confirmed: since 1 July 2024, the List Management REST API and list skills no longer give third-party skills or apps access to the Alexa Shopping and To-Do lists. Official read/write access is gone. Unofficial integrations (HA Alexa Devices since May 2026, Alexa Media Player, alexa-remote2) still read and write the lists through the Alexa-app API.

### Cited Findings

- Amazon's deprecated-features page: "List skills and the List Management REST API to access Alexa lists, such as the Alexa Shopping and To-Do lists, in your skills or apps are no longer supported", effective July 1, 2024. (Search extract of the official page.) — [Amazon: Deprecated Features](https://developer.amazon.com/en-US/docs/alexa/ask-overviews/deprecated-features.html)
- User impact: "Starting in July, you won't be able to add items to lists on third-party apps like Todoist and AnyList with Alexa voice control through their existing Skills." (The Verge article, read via a repost site, so a secondary copy.) — [Verge article repost](https://rss.boorghani.com/?p=201118)
- The Bring! list app reported that users must now name the app in each command, otherwise items go to Alexa's built-in list. — [Galaxus](https://galaxus.be/en/page/alexa-amazon-makes-life-difficult-for-suppliers-like-bring-33803)
- The HA core Alexa Devices integration now has a To-do list platform covering "Shopping, to-do, and custom lists", and list changes arrive by push events. — [HA: Alexa Devices](https://www.home-assistant.io/integrations/alexa_devices/) ([source](https://github.com/home-assistant/home-assistant.io/blob/current/source/_integrations/alexa_devices.markdown))
- The underlying library added "Implement todo lists" in aioamazondevices v13.8.0 (2026-05-26) and a todo-parsing fix in v14.0.5 (2026-06-16). — [aioamazondevices CHANGELOG](https://github.com/chemelli74/aioamazondevices/blob/main/CHANGELOG.md)
- alexa-remote2 v8.0.0 (2025-07-13) moved to new "V2" list and smarthome endpoints (`getListsV2`, `getListItemsV2`, `getSmarthomeDevicesV2`) because Amazon changed the APIs, and some changes were breaking. `deleteListItem` now needs the item version. — [Apollon77/alexa-remote README changelog](https://github.com/Apollon77/alexa-remote)

### Inferences

- A dashboard "shopping list" tile tied to Alexa voice adds is possible only through an unofficial route, best the HA Alexa Devices `todo` entity, which the dashboard reads through the HA API. An alternative is to keep the canonical list outside Alexa (e.g. HA's local to-do list) and give up "Alexa, add milk".
- Amazon changed the list endpoints at least once (mid-2025, per alexa-remote2), so list support is a recurring break point.

### Gaps

- I found no Amazon statement on whether list access will ever return for Alexa+ developers.

## Q3. Unofficial routes: Alexa Media Player (HACS), HA core "Alexa Devices", Node-RED alexa-remote2; status, login/2FA, maintenance, Alexa+ breakage

### Takeaway

As of October 2026 all three are actively maintained, with releases in the last 1–8 weeks. HA core **Alexa Devices** (since HA 2025.6, built on `aioamazondevices`, quality scale "platinum") now covers most of what people used Alexa Media Player for: speak/announce notify entities, a media player with controls, routine buttons, timer/alarm/reminder sensors, DND, to-do lists, and push updates over HTTP/2. **Alexa Media Player** (HACS) is still released often but has many reports in 2026 of logins lost after HA updates. Both need an Amazon account with authenticator-app (TOTP) 2FA and hit Amazon rate limits. I found no source tying a breakage specifically to Alexa+.

### Cited Findings

#### HA core Alexa Devices

- Metadata: `ha_release: '2025.6'`, `ha_iot_class: Cloud Polling`, `ha_quality_scale: platinum`, codeowners @chemelli74 and @jamesonuk. Platforms: binary_sensor, button, event, media_player, notify, select, sensor, switch, todo. — [HA: Alexa Devices](https://www.home-assistant.io/integrations/alexa_devices/) ([source](https://github.com/home-assistant/home-assistant.io/blob/current/source/_integrations/alexa_devices.markdown))
- Supported devices: Echo Auto, Dot, Flex, Plus, Show, Fire TV Stick, Fire Tablet, Air Quality Monitor, and third-party devices with Alexa built in. — same source
- Features: "Button – Execute Alexa routines, restart the device"; "Media Player – Play audio/video from several sources"; "Notify – Speak and Announce" (Announce plays the chime first; messages accept SSML); "Select – Drop In status"; "Switch – Do not disturb"; "To-do list – Shopping, to-do, and custom lists". It also has timestamp sensors for the next alarm, timer and reminder with labels; temperature, illuminance and connectivity sensors; and an event entity per device with the voice command, Alexa's reply and the Voice ID first name. — same source
- SSML audio must be served over HTTPS and meet size and bit-rate limits, so "you will not be able to play music this way". — same source
- Data updates: polls every 5 minutes. To-do lists, voice history, volume, media state and DND arrive by push. — same source
- Login: "This integration requires multi-factor authentication using an authentication app"; the account must use app-based 2FA as the preferred method, not SMS. The config flow takes email, password and an OTP. — same source
- Known limits: Amazon rate limiting (`Too many requests for path ['listEndpoints']`, "Failed to refresh communications settings"). Reminders may be missing for accounts in an Alexa Household. Some sensors only update if they are used in an Alexa routine. — same source
- Library cadence (`aioamazondevices`, PyPI): latest v16.8.1 on 2026-10-15, after v16.7.0 and v16.8.0 on 2026-10-14. Changelog milestones: "Media player controls" v13.1.0 (2026-03-14), "Add routines support" v13.3.0 (2026-03-24), "Add server-sent-events via http2" v13.5.0 (2026-05-07), "Implement todo lists" v13.8.0 (2026-05-26), "Add smart home push events" v16.4.0 (2026-10-11). — [PyPI: aioamazondevices](https://pypi.org/project/aioamazondevices/); [CHANGELOG](https://github.com/chemelli74/aioamazondevices/blob/main/CHANGELOG.md)
- An older forum post says the native integration had no free-text TTS service, only `send_sound`. Current docs contradict it (Speak and Announce notify entities exist), so the post is outdated. — [HA community: Alexa Devices media player feedback](https://community.home-assistant.io/t/alexa-devices-media-player-feedback/1012636/10); contradicted by [HA: Alexa Devices](https://www.home-assistant.io/integrations/alexa_devices/)
- A setup failure on HA 2026.6.0 ("Cannot find account owner customer ID") was fixed for at least one user by re-entering the password and OTP. — [HA community thread](https://community.home-assistant.io/t/alexa-devices-failed-setup-will-retry-error-retrieving-data-cannotretrievedata-cannot-find-account-owner-customer-id/1012849)

#### Alexa Media Player (alandtse, HACS)

- Releases: v5.17.0 (2026-10-16, "Latest"), v5.16.2 (2026-10-12), v5.16.1 (2026-09-13), v5.16.0 and v5.15.8 (2026-09-12), v5.15.7 (2026-07-23), v5.15.6 (2026-07-02), v5.15.5 (2026-06-26), v5.15.4 (2026-05-22), v5.15.3 (2026-05-19). Recent fixes include an `AlexaLogin.login()` timeout, reauth handling, a 2026.7 blocking-call fix, HTTP/2 handlers and Amazon Kids support. No release mentions Alexa+ or deprecation. — [GitHub releases](https://github.com/alandtse/alexa_media_player/releases)
- Backend library `AlexaPy`: v1.32.0 (2026-10-16), v1.31.0 (2026-09-16), v1.30.1 (2026-09-13). — [PyPI: AlexaPy](https://pypi.org/project/AlexaPy/) (repo on GitLab: keatontaylor/alexapy)
- README features: play/pause/stop, next/previous, volume, and now-playing title, artist, album and album image. — [README](https://github.com/alandtse/alexa_media_player)
- 2026 reports of lost logins: on HA 2026.7.1 with AMP 5.15.6, auth was lost after every HA restart and fixed by re-running login with a fresh OTP ([simon42 forum, German](https://community.simon42.com/t/alexa-media-player-verliert-seit-heute-nach-jedem-home-assistant-neustart-die-authentifizierung-ha-core-2026-7-1-amp-5-15-6/89025)). After HA 2026.7.4, announcements stopped even after reconfiguring, and a commenter suggested moving to core Alexa Devices ([HA community](https://community.home-assistant.io/t/alexa-announcements-broken-after-2026-7-4/1019791)). On 2026.6.3, devices showed unavailable, and one user argued AMP is "heading towards EOL" ([HA community](https://community.home-assistant.io/t/alexa-media-player-unavailable/1014443)). A similar login failure appeared after HA 2025.11.0 ([GitHub issue #3087](https://github.com/alandtse/alexa_media_player/issues/3087)).

#### Node-RED / Node.js (alexa-remote2)

- `alexa-remote2` (Apollon77) latest v8.1.1, 2026-08-15 ("Alexa-Cookie fixes for 2FA and redirects during Logins"). v8.1.0 (2026-07-06) added a `NotifyNowPlayingUpdated` event, v8.0.4 (2025-11-06) "Adjusts Authentication check to recent Amazon changes", and v8.0.0 (2025-07-13) brought the breaking V2 list and smarthome APIs. Login uses a local cookie-capture proxy, and the README warns about an "alexa.amazon.xx is deprecated" page during login. — [npm: alexa-remote2](https://www.npmjs.com/package/alexa-remote2); [GitHub README](https://github.com/Apollon77/alexa-remote)
- The library has methods for smart home devices (`getSmarthomeDevicesV2`, `querySmarthomeDevices`, `setEnablementForSmarthomeDevice`) and routines (`executeAutomationRoutine`). — [Apollon77/alexa-remote README](https://github.com/Apollon77/alexa-remote)
- Node-RED node `node-red-contrib-alexa-remote2-applestrudel` latest v5.1.1, published 2026-07-08. — [npm](https://www.npmjs.com/package/node-red-contrib-alexa-remote2-applestrudel)

#### Alexa+

- None of the AMP releases from May to October 2026 mention Alexa+ ([releases](https://github.com/alandtse/alexa_media_player/releases)). Searches found no report attributing an Alexa Devices, AMP or alexa-remote2 breakage specifically to Alexa+ ([search: HA community threads above](https://community.home-assistant.io/t/alexa-announcements-broken-after-2026-7-4/1019791)).

### Inferences

- For a Home Assistant-backed dashboard, core **Alexa Devices** is now the default choice. It ships with HA, is platinum quality, has push updates and covers announce, media, routines and lists. AMP is worth it only for a feature Alexa Devices lacks. Running both doubles exposure to Amazon rate limits.
- Expect periodic re-authentication (password plus TOTP). The dashboard should show an "Alexa disconnected" state rather than failing silently. Use a dedicated or main Amazon account with authenticator-app 2FA; SMS-only 2FA will not work with Alexa Devices.
- These integrations break when Amazon changes the Alexa-app backend, not on a fixed schedule. Fast maintainer response (multiple releases per month in 2026) is the main mitigation. All of this violates no published API contract only because none exists; Amazon could block it at any time, and AMP's own README says so.
- Controlling Alexa-linked lights or plugs through these integrations is at best partial. Alexa Devices' docs list no light or switch platform for Alexa-linked endpoints. The October 2026 "smart home push events" commit suggests work in that direction but is not documented as a feature. alexa-remote2 exposes smart-home device methods, so a Node-RED flow could probably toggle Alexa-linked devices, but that is the most fragile route.

### Gaps

- I could not map each aioamazondevices milestone (media player, routines, to-do) to the HA release that first shipped it. The HA release notes could not be fetched.
- AMP's full current feature list (wiki) was not read. The README is minimal.
- I found no authoritative statement on whether Echo devices upgraded to Alexa+ behave differently for these APIs (e.g. "last called", TTS).

## Q4. How to control lights/plugs that exist only in the Alexa app from another system

### Takeaway

Ranked by robustness:

1. **Matter multi-admin.** If the device was added to Alexa via Matter, share it from the Alexa app ("Other Assistants and Apps" → "Add another" → 11-digit code) into HA's Matter fabric. Control is then local and supported.
2. **Native vendor app or cloud.** Add the device in the vendor's own app and use the vendor's HA integration or API. This also works with Alexa kept in parallel.
3. **Re-pair.** Factory-reset the device and pair it to HA directly (Matter, Zigbee, Wi-Fi local).
4. **Trigger Alexa routines from outside**, via Alexa Devices routine buttons, or by exposing HA virtual sensors to Alexa as routine triggers.
5. **Unofficial alexa-remote2 smart-home calls.**

Amazon-branded "Works exclusively with Alexa" devices (older Amazon Smart Plug or bulbs) can only be reached through routes 4 and 5, or replaced. Newer Amazon Basics plugs are Matter-certified.

### Cited Findings

- Amazon's help page "Set Up an Alexa-connected Matter Device to Another Voice Assistant" documents sharing an Alexa-connected Matter device to another ecosystem. — [Amazon Customer Service](https://www.amazon.com/gp/help/customer/display.html?nodeId=T6iLWTtbZygaBJnQhg)
- Steps (TP-Link): in the Alexa app, open the device's Settings → "Other Assistants and Apps" → "Add Another" to generate an 11-digit setup code. The code is single-use. Pages disagree on expiry: older TP-Link pages say 3 minutes, a newer one says 15. Phone, device and hub must be on the same Wi-Fi, and the device needs 2.4 GHz. — [TP-Link FAQ 3573](https://www.tp-link.com/us/support/faq/3573/); [TP-Link FAQ 3564](https://www.tp-link.com/us/support/faq/3564/); [Tapo FAQ 299](https://www.tapo.com/faq/299)
- HA side: Settings → Connectivity → Matter → Add device → "Yes. It's already in use." HA is a Matter controller only, "not a bridge itself", and cannot turn HA devices into Matter devices. Matter devices should support at least 5 fabrics. HA notes that a native integration such as Hue can offer more than the Matter route. — [HA: Matter](https://www.home-assistant.io/integrations/matter/) ([source](https://github.com/home-assistant/home-assistant.io/blob/current/source/_integrations/matter.markdown))
- HA lists "some newer Amazon Echo devices" as Matter controllers. — [HA: Matter](https://www.home-assistant.io/integrations/matter/)
- Community reports say Echo Thread border routers do not let HA send commands through their Thread network; Thread credentials stay locked to the vendor. One user reports that, after sharing to HA, a device can be removed from Alexa without effect (anecdotal). — [HA community: Echo 4th gen Matter/TBR](https://community.home-assistant.io/t/amazon-echo-4gen-matter-devices-thread-border-router/567706); [HA community: My first Matter device](https://community.home-assistant.io/t/my-first-matter-device-i-dont-understand/784325)
- The Amazon Basics Smart Indoor Plug is CSA Matter-certified ("works with Amazon Alexa, Apple Home and Google Home through Matter"), with 2.4 GHz Wi-Fi only. One German forum user could not commission it into HA Matter, unresolved. 9to5Mac (2026-07-25) covered the Amazon Basics Matter plug working with Apple Home. — [CSA product listing](https://csa-iot.org/csa_product/amazon-basics-smart-indoor-plug-single-outlet/); [ComputerBase forum](https://www.computerbase.de/forum/threads/amazon-basic-matter-wi-fi-plug-home-assistant-matter-integration.2263407/); [9to5Mac](https://9to5mac.com/2026/07/25/the-amazon-basics-matter-plug-delivers-apple-home-control-for-under-10/)
- Triggering Alexa routines from HA:
  - Alexa Devices exposes Alexa routines as HA **Button** entities ("Execute Alexa routines"). — [HA: Alexa Devices](https://www.home-assistant.io/integrations/alexa_devices/)
  - With the Alexa Smart Home skill (Nabu Casa or manual) and Proactive Events, HA binary sensors exposed as contact or motion sensors can trigger Alexa routines. Switches and input_booleans appear to Alexa as contact sensors (on = Open). Buttons and input_buttons appear with "presence detection" capability and trigger via the "Person" option. — [HA: Alexa Smart Home](https://www.home-assistant.io/integrations/alexa.smart_home/) ([source](https://github.com/home-assistant/home-assistant.io/blob/current/source/_integrations/alexa.smart_home.markdown))
- Emulated Hue: HA can pose as a Philips Hue bridge so an Echo controls HA entities locally ("everything looks like a light bulb"). New Alexa devices need `listen_port: 80`, the setup is tied to the Echo it was discovered with, and the `type` option is deprecated. — [HA: Emulated Hue](https://www.home-assistant.io/integrations/emulated_hue/) ([source](https://github.com/home-assistant/home-assistant.io/blob/current/source/_integrations/emulated_hue.markdown))

### Inferences

- First triage step: list each light or plug's brand and protocol. Most third-party Wi-Fi lights and plugs (Kasa/Tapo, Govee, Wiz, Meross and others) were added to Alexa via the vendor's cloud skill. They already have a vendor account and usually an HA integration, so the dashboard can control them without touching Alexa, and Alexa voice keeps working in parallel. Devices added to Alexa via Matter can be shared with a code. Only Amazon-exclusive devices are truly stuck.
- The "virtual switch → Alexa routine" pattern (HA input_boolean exposed to Alexa as a contact sensor, with an Alexa routine "when contact opens, turn on lamp") is a supported, cloud-based workaround for stuck devices. It is one-way and gives no state back to the dashboard. It needs the Alexa Smart Home skill (Nabu Casa, or a manual skill with a public HTTPS endpoint).
- Alexa Devices routine buttons do the same thing with less setup, but over the unofficial API.
- A Matter device shared from Alexa via an Echo Thread border router may not work in HA unless HA has its own Thread border router. Wi-Fi Matter devices avoid this.

### Gaps

- I could not confirm from an Amazon source whether devices linked through a vendor cloud skill (non-Matter) can ever be shared via "Other Assistants and Apps". I expect not, because the option is a Matter multi-admin feature.
- No URL found for the claim that the older Amazon Smart Plug or Amazon Basics bulbs are "Works exclusively with Alexa"; a search extract mentioned an Alexa-only bulb listing without a citable URL. The owner should check device labels.
- The current Matter setup-code expiry on Alexa (3 vs 15 minutes) conflicts between TP-Link pages.

## Q5. Alexa+ status in the US (2026) and whether it changed integration options

### Takeaway

Alexa+ has been available to all US customers since 4 February 2026: free with Prime, US$19.99/month otherwise, plus a limited free chat tier on Alexa.com and the app. Some older Echo hardware stays on classic Alexa. For developers it added AI-native SDKs for businesses, not new home-control or device-control APIs. Smart home skills were renamed "add-ons" and keep working. For a dashboard builder the integration options are unchanged.

### Cited Findings

- "Amazon.com Inc. on Feb. 4, 2026, made its generative AI-powered Alexa+ assistant available to every customer in the United States." — [MarketScreener](https://au.marketscreener.com/news/amazon-rolls-out-alexa-nationwide-in-the-us-ce7e5adadf8df527); [MacRumors, 2026-02-04](https://www.macrumors.com/2026/02/04/amazon-alexa-plus-prime)
- Free for Prime members. Non-Prime users pay $19.99/month or use a limited free chat version on Alexa.com and the Alexa app. Several older Echo, Echo Show, Echo Buds, Echo Spot, Fire TV and Fire tablet models remain on classic Alexa. English (US) only. — [IBTimes AU](https://www.ibtimes.com.au/amazon-rolls-out-alexa-ai-assistant-all-us-users-free-prime-members-nearly-year-after-debut-1861352); [MacRumors](https://www.macrumors.com/2026/02/04/amazon-alexa-plus-prime)
- Alexa+ reportedly became free on eligible US Fire TV devices regardless of Prime from 2026-08-19. Only a deals site reports this; unverified. — [Dealmoon](https://www.dealmoon.com/en/fire-tv-amazon-alexa-plus-now-free-fire-tv-users-benefit-from-updated-policy/6075780.html)
- Developer side: Alexa AI Action SDK, Web Action SDK and Multi-Agent SDK for partners ([Amazon developer blog, Feb 2025](https://developer.amazon.com/en-US/blogs/alexa/alexa-skills-kit/2025/02/new-alexa-announce-blog)). Smart home skills were renamed "smart home add-ons" with existing integrations unchanged ([Amazon docs](https://developer.amazon.com/en-US/docs/alexa/smarthome/understand-the-smart-home-skill-api.html)).
- Reports of an Amazon internal "Moonraker" project for multi-step commands come from leaked-documents coverage and are unconfirmed. — [byteiota](https://byteiota.com/amazon-moonraker-alexa-agentic-developer-guide/)

### Inferences

- Alexa+ makes Alexa better at natural-language control of devices already linked to it. That helps the reverse direction (Q6): HA entities exposed to Alexa can be controlled conversationally. It does not give an external dashboard any new way in.
- Alexa+ changes Amazon's backend, so the unofficial integrations (Q3) face more churn. The heavy 2026 release cadence of aioamazondevices, AlexaPy and alexa-remote2 is consistent with that, but no source states the cause.

### Gaps

- I did not verify whether Amazon retired the alexa.amazon.com web app or changed the Alexa app login flows as part of Alexa+. alexa-remote2's README mentions a "deprecated" page during login but gives no date.
- I could not verify the Fire TV free-tier claim against Amazon's own announcement.

## Q6. Reverse direction: exposing the home's devices to Alexa voice (Nabu Casa, manual Smart Home skill, haaska, emulated Hue)

### Takeaway

There are three practical options with HA:

1. **Home Assistant Cloud (Nabu Casa)**: US$6.50/month or $65/year, a few clicks, no open ports.
2. **Manual Alexa Smart Home skill**: an Amazon developer account plus an AWS Lambda forwarding to HA, which needs a public HTTPS endpoint on port 443 with a trusted certificate.
3. **Emulated Hue**: local, lights and switches only, needs port 80.

haaska is an older Lambda bridge that the HA docs' own manual Lambda method has made unnecessary. Its README is still live, but I did not check its activity.

### Cited Findings

- HA: the manual Smart Home skill "takes considerable effort to configure. Your Home Assistant instance must be accessible from the Internet, and you need to create an Amazon Developer account and an Amazon Web Services (AWS) account. An easier solution is to use Home Assistant Cloud." Requirements: HTTPS on port 443 with a publicly trusted certificate (self-signed won't work), and AWS Lambda in specific regions (free tier up to 1M requests). — [HA: Alexa Smart Home](https://www.home-assistant.io/integrations/alexa.smart_home/) ([source](https://github.com/home-assistant/home-assistant.io/blob/current/source/_integrations/alexa.smart_home.markdown))
- Supported entity types include alarm panels, binary sensors (routine triggers), buttons (scenes), cameras, climate, covers, event and doorbell entities, fans, lights and more. Proactive Events enable state reporting and routine triggers. — same source
- Nabu Casa pricing: US$6.50/month or $65/year excluding sales tax, with a 30-day free trial. The Alexa setup docs are on nabucasa.com. — [Nabu Casa pricing](https://www.nabucasa.com/pricing/); [Nabu Casa Alexa config](https://www.nabucasa.com/config/amazon_alexa/)
- haaska: "implements a bridge between the Home Assistant Smart Home API and the Alexa Smart Home Skill API", MIT-licensed, maintained by mike-grant and contributors. — [mike-grant/haaska](https://github.com/mike-grant/haaska)
- Emulated Hue: local, makes HA entities look like Hue lights, needs `listen_port: 80` for new Echos, and is tied to the Echo used for discovery. — [HA: Emulated Hue](https://www.home-assistant.io/integrations/emulated_hue/)

### Inferences

- For a technical owner, the manual skill is feasible but adds a Lambda, an Amazon developer account, a public HTTPS endpoint and certificate renewal to look after. Nabu Casa's $65/year buys that away and also provides remote access, which other researchers on the hub architecture may count.
- Exposing HA virtual switches to Alexa also unlocks the routine-trigger workaround in Q4. One subscription covers both needs.

### Gaps

- haaska's last commit or release date could not be retrieved (GitHub API blocked here). Treat it as legacy.
- The rumour that Nabu Casa will rename the service in December 2026 is unverified. It appeared in one low-quality search result, which I did not cite.

## Q7. Sonos local control: UPnP/SOAP on port 1400, events, libraries, HA integration; did the 2024 app rewrite or 2025–2026 firmware restrict it?

### Takeaway

Local UPnP/SOAP control on TCP 1400 still works in October 2026. The HA Sonos integration is "Local Push", and SoCo shipped v0.32.0 on 2026-10-15. There are two real changes. First, the May 2024 app rewrite moved Sonos's own app off UPnP/SOAP. Second, a July 2025 player update added Connection Security settings: a **UPnP** toggle (default ON; turning it off breaks HA, SoCo and the like with HTTP 403) and an **Authentication** toggle (default OFF; when ON, third-party integrations using Sonos's cloud and LAN APIs must authenticate). Sonos now calls UPnP "unsupported" and "no longer actively maintained", which is a clear long-term risk signal but not a removal.

### Cited Findings

#### Platform changes

- Sonos announced the rebuilt app on 2024-04-23 (global 2024-05-07, all S2 products, plus a web app). Desktop apps are being sunset. — [Business Wire press release](https://www.businesswire.com/news/home/20240423695150/en); [iPhone in Canada](https://www.iphoneincanada.ca/2024/04/23/sonos-unveils-all-new-app/)
- A Sonos staffer in the redesign community threads said that until the new update, "the app was using an older set of technology based on UPnP and SOAP". Users asked whether the new local API would be documented, and I found no official answer. (Search extract.) — [Sonos community: app redesign AMA](https://en.community.sonos.com/owners-cafe-228997/sonos-app-redesign-ama-6893676/index57.html)
- Connection Security settings (Sonos support): UPnP "pertains to integrations that utilize the unsupported UPnP protocol" and its default is ON. Authentication applies to third-party integrations using Sonos cloud and LAN APIs and requires them to authenticate. By default, integrations keep working as today. (Search extract of the official page.) — [Sonos support: Adjust Connection Security settings](https://support.sonos.com/en-us/article/adjust-connection-security-settings)
- The July 2025 player update (around 2025-07-08) added the Authentication, UPnP and Guest Access settings. Authentication is off by default; UPnP and Guest Access are on. UPnP is described as support for the older protocol, "no longer actively maintained". Disabling UPnP also stops the Sonos macOS/Windows apps controlling the system. (Search extract.) — [Sonos community: "A new player update is live!"](https://en.community.sonos.com/product-updates/a-new-player-update-is-live-6930092)
- Third-party effect: Brilliant warns that turning on "Require Authentication" "will prevent your Brilliant devices and mobile app from being able to control Sonos". — [Brilliant support](https://support.brilliant.tech/hc/en-us/articles/38086493441819-Sonos-Connected-Products-Settings-Authorization)
- The newer local "LAN API" used by Sonos's own apps has no public documentation. Forum posters say it needs a client certificate (claim, not official). — [Sonos community: Where is the LAN API documented?](https://en.community.sonos.com/advanced-setups-229133/where-is-the-lan-api-documented-6930288)

#### Home Assistant Sonos integration

- `ha_iot_class: Local Push`, discovered by SSDP and zeroconf, platforms media_player, number, select, sensor, switch, button and binary_sensor, quality scale bronze. Also supports IKEA Symfonisk. — [HA: Sonos](https://www.home-assistant.io/integrations/sonos/) ([source](https://github.com/home-assistant/home-assistant.io/blob/current/source/_integrations/sonos.markdown))
- Prerequisite: "UPnP must be enabled on your Sonos system for this integration to work. In the Sonos app, go to Account > Privacy and Security > UPnP". With UPnP off, setup fails with `403 Client Error: Forbidden for url: http://<ip>:1400/DeviceProperties/Control`. — same source
- Network: speakers must reach the HA host on TCP 1400 for push events (HA tries up to 100 ports above 1400 if it is busy), otherwise HA falls back to slower polling. Announce/TTS needs TCP 1443 on each speaker reachable from HA. `advertise_addr` is available for Docker/NAT. — same source
- Features: per-alarm switches, bass/treble/loudness, home-theater settings, battery sensors, a favorites sensor, `play_media` with Spotify/Tidal links, Plex, Sonos music library, playlists, enqueue modes, `announce: true` overlay with volume, and `media_player.search_media` on the local library. Some state (TV Autoplay) is polled every 30 s because UPnP emits no events for it. — same source
- HA added detection for UPnP being disabled in 2025.12.5. A user found UPnP disabled because "it wasn't necessary when I set it up a few months ago". (Search extract of the issue.) — [home-assistant/core issue #156328](https://github.com/home-assistant/core/issues/156328)
- A July 2025 discovery outage (HA 2025.7.x) was traced to an `async-upnp-client` bump, not to Sonos. — [HA community: Sonos integration fails to detect devices](https://community.home-assistant.io/t/sonos-integration-fails-to-detect-devices/911767)

#### Libraries (release dates from registries, fetched 2026-10-17)

- **SoCo (Python)**: v0.32.0 on 2026-10-15 (Beam Ultra and Amp Multi support, queue reordering, favorites write API, EventListener fixes, Python 3.15 in CI); v0.31.5 on 2026-10-06; v0.31.0 in April (Arc Ultra, experimental surround pairing, cached ZoneGroupState). — [PyPI: soco](https://pypi.org/project/soco/); [SoCo releases](https://github.com/SoCo/SoCo/releases)
- SoCo builds an absolute album-art URL from the track's UPnP `albumArtURI` (`get_current_track_info` → `build_album_art_full_uri`). — [SoCo core.py](https://github.com/SoCo/SoCo/blob/master/soco/core.py); [music_library.py](https://github.com/SoCo/SoCo/blob/master/soco/music_library.py)
- **node-sonos** (`sonos` on npm): latest v1.14.3, published 2026-02-27. — [npm: sonos](https://www.npmjs.com/package/sonos); [bencevans/node-sonos](https://github.com/bencevans/node-sonos)
- **node-sonos-ts** (`@svrooij/sonos`): latest v2.5.0, published 2022-06-16 (package metadata modified 2026-07-08). Effectively stale. — [npm: @svrooij/sonos](https://www.npmjs.com/package/@svrooij/sonos); [svrooij/node-sonos-ts](https://github.com/svrooij/node-sonos-ts)
- **node-sonos-http-api** (jishi): not on npm. The last master commits are 2025-03-22 (keep-alive fix) and 2025-01-29 (README), 541 commits, 177 open issues, not archived. The README says it "still works" with S2 but "might break in the future" if Sonos drops UPnP. It has an HTTP REST bridge with `/zones`, `/{room}/state` (incl. `albumArtURI`), grouping (join/leave), presets, webhooks and Server-Sent Events at `/events`. — [jishi/node-sonos-http-api](https://github.com/jishi/node-sonos-http-api); [commits](https://github.com/jishi/node-sonos-http-api/commits/master)
- **node-red-contrib-sonos-plus**: v6.9.5, 2026-03-16. — [npm](https://www.npmjs.com/package/node-red-contrib-sonos-plus)
- **sonos-websocket** (jjlawren, Python): an async library for the local websocket API, mainly for audio clips (overlay with ducking). The openHAB JRule Sonos project says the local websocket is on TCP 1443 and S1 speakers do not support the AudioClip websocket API. — [jjlawren/sonos-websocket](https://github.com/jjlawren/sonos-websocket); [PyPI: sonos-websocket](https://pypi.org/project/sonos-websocket); [seaside1/jrule-sonos](https://github.com/seaside1/jrule-sonos)
- **Java**: the only Java Sonos project surfaced was the openHAB JRule Sonos rule project above. I found no maintained standalone Java UPnP library in this session. — [seaside1/jrule-sonos](https://github.com/seaside1/jrule-sonos)

### Inferences

- Local UPnP is the best route for a dashboard today: no cloud account, sub-second push events, full grouping and queue control, and album art. Its risk is policy, not technical. Sonos labels it unsupported and has added a kill switch, so a future firmware could turn it off by default or remove it. Keep the Sonos code behind an adapter so a switch to the cloud Control API (Q8) is contained.
- The owner must leave UPnP ON and Authentication OFF in the Sonos app (Account → Privacy and Security). The dashboard's health check should treat HTTP 403 on `:1400/DeviceProperties/Control` as "UPnP disabled".
- For a Node.js backend, `sonos` (node-sonos) is the maintained library. jishi's HTTP API is convenient but semi-dormant. node-sonos-ts is stale. For Python, SoCo is active. With HA, the HA Sonos integration (built on SoCo) already handles discovery, events and announcements.
- The ADR-0005 Java candidate has no strong Sonos library. A Java backend would either call HA's API, call a sidecar (node-sonos-http-api or SoCo), or implement the few SOAP calls itself.

### Gaps

- I could not open Sonos's support or community pages directly, so the exact defaults and dates come from search extracts, which agree with each other.
- I found no statement from Sonos on a timeline to remove UPnP or to publish the LAN API. I also found no report of a 2026 firmware breaking UPnP control.
- Whether the "Authentication" toggle, when ON, also blocks UPnP on port 1400 (rather than only the cloud and LAN APIs) is not stated in the extracts.

## Q8. Sonos official cloud Control API (developer.sonos.com / docs.sonos.com)

### Takeaway

The Control API is a supported, documented JSON API at `api.ws.sonos.com/control/api/v1`. It uses OAuth 2.0 authorization-code flow with a client key and secret from the Sonos developer "integration manager". It covers households, groups, playback, metadata, group and player volume, favorites, playlists and audio clips. Events come only by webhook to a **public HTTPS callback URL** registered per API key. Rate limits exist (HTTP 429) but no numbers are published. For a home dashboard it works but is clumsier than local UPnP: you need a public HTTPS endpoint for events (or polling), plus token refresh, plus cloud latency.

### Cited Findings

- "The Sonos Control API is a JSON-based application protocol for controlling audio playback on Sonos players." The integration flow is authorize → discover → control → subscribe. All players belong to groups (a single player is a group of 1). — [Sonos docs: Control](https://docs.sonos.com/docs/control); [About Control API](https://docs.sonos.com/reference/about-control-api); [Control Sonos Players](https://docs.sonos.com/docs/control-sonos-players)
- Endpoint examples: `GET/POST .../groups/{groupId}/groupVolume`, `POST .../groupVolume/subscription`, and `POST .../groups/{groupId}/favorites` (loadFavorite). — [groupVolume subscribe](https://docs.sonos.com/reference/groupvolume-subscribe-groupid); [favorites loadFavorite](https://docs.sonos.com/reference/favorites-loadfavorite-groupid)
- Namespaces include groups, groupVolume, playback, playbackMetadata, favorites and playlists, with a single `playback-control-all` OAuth scope. (Third-party summary, not confirmed against Sonos's own pages.) — [apis.io: Sonos](https://apis.io/providers/sonos/)
- `audioClip` (`loadAudioClip`, with priority rules) is documented for the cloud API. — [Sonos docs: audioClip](https://docs.sonos.com/reference/audioclip)
- Registration: in the integration manager, click "New control integration", give a name, description and category (the name is shown to users on the Sonos login consent screen), then create a key. "Sonos uses client credential keys to identify integrations." Auth requests use `client_id` and `response_type=code`, and the code is exchanged with key and secret via HTTP Basic auth. — [Sonos docs: Authorize](https://docs.sonos.com/docs/authorize); [Add app authentication](https://docs.sonos.com/docs/add-app-authentication); [Sonos community: OAuth2 blog](https://en.community.sonos.com/controllers-and-music-services-228995/oauth2-sonos-blog-6857003)
- Redirect URI: the official docs say it must be publicly routable HTTPS. This conflicts with `sonos-cli`, which uses `http://localhost:5000`, and with a 2018 PHP project saying "Localhost is not accepted". — [Sonos docs: Authorize](https://docs.sonos.com/docs/authorize); [PyPI: sonos-cli](https://pypi.org/project/sonos-cli/0.3.8b1/); [hubwareHouse/sonos](https://github.com/hubwareHouse/sonos)
- Events: "in the integration manager, you register your callback URL for each API key. Your callback URL must use HTTPS", separate from the redirect URI. A subscribe call returns 200 if a callback is registered, 403 if not. Events are HTTPS POSTs with a signature header to verify with the key and secret. — [Sonos docs: Subscribe](https://docs.sonos.com/docs/subscribe)
- A forum reply says no real HTTPS callback is needed for OAuth alone (a custom scheme works), "Later on you DO need a real https callback if you want to use cloud events." — [Sonos community](https://en.community.sonos.com/controllers-and-music-services-228995/oauth2-sonos-blog-6857003)
- Rate limiting: HTTP 429 "The number of requests have hit the rate limit". Send a User-Agent ("If you don't provide a User-Agent, we may have to throttle large spikes"). No numeric limits are published in the extracts. — [Sonos docs: Control](https://docs.sonos.com/docs/control)
- Prior art: `jjlawren/sonos_cloud` is an HA custom integration using the cloud API for better TTS and alerts. openHAB users used cloud `audioClip` the same way. — [jjlawren/sonos_cloud](https://github.com/jjlawren/sonos_cloud); [openHAB community](https://community.openhab.org/t/sonos-announcements-notifications-via-audioclip-sonos-cloud/147601)

### Inferences

- A personal integration appears to need only a free developer account plus a control integration and key. Nothing in the extracts mentions an approval step for use on your own household, though public distribution likely has terms or certification (not verified). The owner authorizes their own Sonos account through OAuth.
- For a LAN-only tablet dashboard, the webhook requirement is the main drawback. It needs a public HTTPS endpoint (e.g. a tunnel or reverse proxy, or HA's remote URL), or else polling with rate-limit risk. The cloud API is a good fallback if Sonos ever disables UPnP, not the first choice.

### Gaps

- Numeric rate limits, token lifetimes and the Sonos developer terms (e.g. restrictions on personal or non-commercial use, branding) could not be retrieved. docs.sonos.com was blocked and the extracts had no numbers.
- I could not confirm from Sonos's own pages that `playbackMetadata` includes an album-art image URL, or check its exact field names.
- I did not determine whether Sonos offers a documented local (LAN) variant of the Control API for third parties. Forum evidence suggests the LAN API is undocumented and may need a client certificate.

## Q9. Showing now-playing album art and controlling grouping from a tablet page: which route is easiest?

### Takeaway

The easiest robust route is **through Home Assistant**. The HA Sonos integration (local push) and Alexa Devices or AMP keep state current. The dashboard subscribes to HA's WebSocket API for `media_player` state (title, artist, album, artwork via HA's proxied entity picture) and calls HA services for play, pause, volume and grouping. Without HA, use node-sonos (Node) or SoCo (Python) on the home server and relay state to the page over WebSocket or SSE. The Sonos cloud API is the most work.

### Cited Findings

- HA Sonos is local push. Speakers push changes to HA on TCP 1400, and polling is the fallback. Sonos supports "standard media player actions". — [HA: Sonos](https://www.home-assistant.io/integrations/sonos/)
- UPnP track info carries `albumArtURI`, which SoCo resolves to an absolute URL on the speaker. jishi's HTTP API also returns `albumArtURI` in `/{room}/state` and offers SSE at `/events` and webhooks. — [SoCo core.py](https://github.com/SoCo/SoCo/blob/master/soco/core.py); [jishi/node-sonos-http-api](https://github.com/jishi/node-sonos-http-api)
- Alexa Media Player exposes Echo now-playing song, artist, album and album image. Alexa Devices pushes media state. — [AMP README](https://github.com/alandtse/alexa_media_player); [HA: Alexa Devices](https://www.home-assistant.io/integrations/alexa_devices/)
- The Sonos cloud API models everything as groups and supports subscriptions per group namespace, but only through a public HTTPS webhook. — [Sonos docs: Control](https://docs.sonos.com/docs/control); [Subscribe](https://docs.sonos.com/docs/subscribe)

### Inferences

- I did not re-verify these in this session: HA's `media_player.join`/`unjoin` actions and `group_members` attribute (which the Sonos integration supports), and HA serving artwork through `/api/media_player_proxy/...` on the HA origin. HA's proxy avoids mixed-content and CORS problems when the dashboard is served over HTTPS, because Sonos art is plain `http://<speaker>:1400/...`. Without HA, the backend should proxy or cache album art for the same reason.
- Grouping UX: model the page around Sonos "groups with a coordinator". Tapping a room joins it to the current coordinator, and long-press unjoins.
- Echo devices cannot join Sonos groups, and Alexa multi-room groups are not controllable through UPnP. Show the two systems as separate players.

### Gaps

- I did not confirm the exact HA artwork-proxy and join/unjoin documentation in this session; the media_player docs source restructured its actions into an include file.

## Q10. Practical recommendation for this owner

### Takeaway

Run Home Assistant on the home server and have the tablet page talk only to HA's WebSocket/REST API.

- **Sonos:** core HA Sonos integration (local UPnP, push). Keep UPnP ON and Authentication OFF in the Sonos app. Treat the Sonos cloud Control API as a documented fallback behind an adapter.
- **Alexa:** core HA **Alexa Devices** (announce/speak, Echo media state and controls, routine buttons, timers/alarms, Alexa shopping list), set up with TOTP 2FA and accepted as unofficial and occasionally re-auth-requiring.
- **Lights and plugs:** move them off "Alexa-only". Use each vendor's own integration or Matter multi-admin share into HA, keeping Alexa voice via Nabu Casa or emulated Hue. Use Alexa routines triggered from HA only for devices that cannot be moved.

### Cited Findings

- Alexa Devices is core HA since 2025.6, platinum quality, with notify, media player, routines, to-do lists and push updates, and requires app-based 2FA. — [HA: Alexa Devices](https://www.home-assistant.io/integrations/alexa_devices/)
- AMP has frequent 2026 releases alongside repeated re-auth or breakage reports, and community advice to move to core Alexa Devices. — [AMP releases](https://github.com/alandtse/alexa_media_player/releases); [HA community](https://community.home-assistant.io/t/alexa-announcements-broken-after-2026-7-4/1019791)
- There is no official API for controlling Alexa from outside, and list APIs ended July 2024. — [Amazon: Deprecated Features](https://developer.amazon.com/en-US/docs/alexa/ask-overviews/deprecated-features.html); [Amazon: Smart Home add-ons](https://developer.amazon.com/en-US/docs/alexa/smarthome/understand-the-smart-home-skill-api.html)
- HA Sonos is local push with UPnP required; the UPnP toggle defaults ON, and Sonos calls UPnP unsupported. — [HA: Sonos](https://www.home-assistant.io/integrations/sonos/); [Sonos support](https://support.sonos.com/en-us/article/adjust-connection-security-settings)
- Matter multi-admin sharing from Alexa to HA is documented by Amazon and HA. — [Amazon help](https://www.amazon.com/gp/help/customer/display.html?nodeId=T6iLWTtbZygaBJnQhg); [HA: Matter](https://www.home-assistant.io/integrations/matter/)
- Nabu Casa US pricing is $6.50/month or $65/year; the manual skill needs public HTTPS on 443 plus AWS. — [Nabu Casa pricing](https://www.nabucasa.com/pricing/); [HA: Alexa Smart Home](https://www.home-assistant.io/integrations/alexa.smart_home/)

### Inferences

- Reliability tiers, for the report: Sonos local is high but carries policy risk. Matter or vendor-native lights are high. The Alexa Smart Home skill (voice into HA) is high and officially supported. Alexa Devices and AMP are medium: unofficial, with periodic re-auth and rate limits. alexa-remote2 smart-home control of Alexa-only devices is low.
- Features that should degrade gracefully on the dashboard: Echo now-playing, the Alexa shopping list, Alexa timers, and announcements to Echo. Announcements can fall back to Sonos `announce` TTS, which is local and more reliable.
- Before deciding, the owner should inventory each Alexa-linked light or plug (brand, protocol, Matter yes/no). That decides whether the "move off Alexa" step is trivial or needs replacement hardware.
- This fits the project's open ADR-0005 (language undecided). Integrating through HA's API keeps the dashboard language-agnostic and avoids depending on the weak Java Sonos and Alexa library ecosystem.

### Gaps

- Amazon's terms of use for automated access to the Alexa-app API were not reviewed. Unofficial integrations likely breach them in spirit, but I found no enforcement reports against individual users.
- No measured reliability data (e.g. mean time between re-auths) exists for Alexa Devices or AMP. The evidence is forum anecdotes.
