---
status: concluded
date: 2026-10-17
authors: [Claude Code]
related: [2026-10-17-tablet-home-dashboard-options]
---

# Tablet hardware, kiosk software, mounting/power and web-platform features for an always-on wall dashboard (US, October 2026)

Research date: 2026-10-17.

Method note for the report writer: the research proxy blocked direct fetches of several primary sources (fully-kiosk.com, home-assistant.io, caniuse.com, developer.mozilla.org, support.apple.com, developer.chrome.com, notebookcheck.com, 9to5toys.com, techrepublic.com). For those, the findings below come from search-engine extracts of the cited page, not from a full read. Only github.com was fetched in full. Treat exact prices and version numbers as "verify before purchase". Every claim carries its URL.

## 1. Platform comparison: iPad vs Android vs Amazon vs dedicated wall panels (prices in the US, 2025-2026)

### Takeaway

The Amazon picture changed this month: on 8 October 2026 Amazon replaced the Fire tablets with Android 16 "Alexa Tablets" that include Google Play, starting at $229.99, and stopped producing Fire tablets. That removes the old "cheap Fire HD 10 + Fully Kiosk" default. The realistic choices are now a Samsung Galaxy Tab A11+ or S10 FE (with a user-set 80/85% charge cap and the full Android kiosk ecosystem), an iPad (A16) at $349 (80% cap and the Safari web-app engine, but weaker kiosk control), or an Android-based in-wall panel such as the Sonoff NSPanel Pro (cheap and flush-mounted, but slow, with an old Android 8.1 WebView). The Google Pixel Tablet, whose dock made it a popular choice, was discontinued in September 2026.

### Cited Findings

#### iPad (iPadOS 18/26)

- The iPad (A16), 11-inch Wi-Fi 128 GB, has a regular price of $349. The 256 GB model is $449 and the 512 GB model $649; cellular adds $100. Sales of the 128 GB model at $299 were seen in June 2026 — [iClarified deal trackers, June 2026](https://www.iclarified.com/82634/amazon-discounts-new-102inch-ipad-to-299-deal). Flag: the exact A16 tracker page was not isolated, so verify on apple.com.
- iPadOS has an optional 80% charge limit (Settings > Battery > Battery Health). Charging stops at about 80% and resumes at about 75%. The iPad "will charge to 100 per cent occasionally" to recalibrate its charge estimate. The limit is available on iPad Pro (M4) and later, iPad Air (M2) and later, iPad mini (A17 Pro) and iPad (A16) — [Apple Support 118418](https://support.apple.com/en-gb/118418). Older iPads (for example the 9th/10th-gen iPad, or the iPad mini 6 still sold at $349) do not get it — same source.
- The feature arrived with the 2024 iPad Air/Pro, together with a battery health and cycle-count menu — [heise, 2024](https://heise.de/en/news/New-iPads-Apple-improves-battery-management-9720431.html).
- A user reports the iPad 80% limit "not working, charging past 80" — [Apple Community thread](https://discussions.apple.com/thread/256322563).
- In iOS/iPadOS 26, every site added to the Home Screen opens as a web app by default. The user can untoggle "Open as Web App", and a site's manifest cannot override that choice — [WebKit, News from WWDC25](https://webkit.org/?p=16993); [heise on iPadOS 26 web-app behaviour](https://heise.de/-10749652).
- Guided Access has its own "Display Auto-Lock" timer (Accessibility > Guided Access), separate from Settings > Display & Brightness > Auto-Lock. Kiosk vendors say to set both to "Never" for a wall display and to keep the iPad plugged in — [Kiosk Group support: Enable Auto-Lock](https://support.kioskgroup.com/article/1024-enable-auto-lock); [Meeting Room App: Guided Access kiosk](https://www.meetingroomapp.com/help/kiosk-mode-for-ipad-guided-access/).
- A Guided Access session ends when the iPad restarts: it boots back to the Home Screen. Single App Mode, which persists across reboots, requires Apple Configurator or MDM (device supervision) — [Virtual Front Desk KB](https://www.virtualfrontdesk.com/knowledge-base-article/ipad-kiosk-mode-guided-access); [RosterElf kiosk guide](https://www.rosterelf.com/support/knowledge-base/time-and-attendance/set-up-ipad-in-kiosk-mode). Flag: which vendor guide made the reboot claim was not isolated.
- An old (2018) report says Guided Access kept the screen on past the configured timer — [Apple Community 8433367](https://discussions.apple.com/thread/8433367). Flag: dated.

#### Android tablets (Samsung, Lenovo, Google)

- The Samsung Galaxy Tab A11+ went on sale in the US on 8 January 2026 as Samsung's most affordable US tablet — [SamMobile](https://www.sammobile.com/news/samsungs-galaxy-tab-a11-goes-sale-usa/). Productchart listed it at $249.99 on 10 July 2026 — [Productchart](https://www.productchart.com/tablets/similar_to_50084). A Prime Day 2026 roundup had the 6/128 GB model at $161 (regular $300) and the 8/256 GB at $229 (regular $380) — [9to5Toys, 24 June 2026](https://9to5toys.com/2026/06/24/2026-galaxy-tab-prime-day/).
- The Galaxy Tab S10 FE was listed at $419.99 on 23 September 2026 — [Productchart](https://www.productchart.com/tablets/similar_to_50991). The Wi-Fi 12/256 GB model was listed at $449.99 in June 2026 — [Smartprix](https://smartprix.com/tablets). A Prime Day sale had the 128 GB model at $335, against a "regular" $550 — [9to5Toys](https://9to5toys.com/2026/06/24/2026-galaxy-tab-prime-day/). Conflict: the "regular" prices differ between sources, so check samsung.com.
- Samsung Battery protection (One UI 6.1+) has three modes. **Basic** pauses at 100% and resumes at 95%. **Adaptive** charges overnight with Maximum and switches to Basic before the user usually wakes. **Maximum** stops at 80%. On One UI versions below 6.1, the only option is a fixed 85% cap — [Samsung Support (AU): Battery protection in One UI 6.1](https://www.samsung.com/au/support/mobile-devices/galaxy-battery-protection-feature-in-one-ui-6-1/). Flag: that page is written for phones. A tablet guide gives the older path (Settings > Device care > Battery > "Protect battery", 85%; the display still reads 100%) — [Xataka Móvil (Spanish)](https://www.xatakamovil.com/samsung/como-proteger-bateria-tu-tablet-samsung-limitando-carga-maxima).
- Lenovo Battery Protection Mode is **automatic**. It turns on after about 7×24 h of continuous charging and holds the battery at about 60%. It can be disabled only temporarily, and it re-engages after another such period — [Lenovo Android tablet launch page (Bluestar)](https://reach.bluestarinc.com/eu-lenovo-launch-tablets). On the Tab M10 Plus (3rd Gen) the toggle is under Settings > Battery > Battery optimization — [Vodafone device guide](https://deviceguides.vodafone.co.uk/lenovo/tab-m10-plus-3rd-gen-android-13/getting-started/turn-optimised-battery-charging-on-or-off/). In January 2026, kiosk users reported that a software update turned the mode back on after they had disabled it — [Hexnode forum](https://www.hexnode.com/forums/topic/how-to-disable-battery-protection-mode-on-lenovo-tablets/).
- The Google Pixel Tablet, whose dock made it popular, has been discontinued and is out of stock at major US retailers. Updates continue to June 2028. The Pixel Tablet 2 was reportedly cancelled — [Tech Advisor, 11 Sept 2026](https://www.techadvisor.com/article/3232946/farewell-pixel-tablet-i-barely-knew-you.html); [Droid Life, 10 Sept 2026](https://www.droid-life.com/2026/09/10/pixel-tablet-is-gone/).

#### Amazon (Fire OS, then Vega, then Android "Alexa Tablets")

- On 8 October 2026 Amazon announced the Alexa Tablet 8, 11 and 12 Pro. They run **Android 16 with the full Google Play Store** and a custom Amazon launcher, and they shipped on 14 October in the US. Android 17 is planned for early 2027, with at least four years of updates. Alexa+ needs Prime or a subscription — [TechCrunch](https://techcrunch.com/2026/10/08/amazon-unveils-new-alexa-tablets-with-alexa-and-google-play-store-access/); [AFTVnews](https://www.aftvnews.com/amazon-announces-alexa-tablets-an-all-new-line-running-full-android-with-the-google-play-store/); [Technology.org](https://www.technology.org/2026/10/09/amazon-alexa-tablets-android-google-play-fire/).
- Prices are $229.99 (8"), $329.99 (11") and $499.99 (12 Pro) — [9to5Toys](https://9to5toys.com/2026/10/08/amazon-debuts-all-new-android-alexa-tablets-today/). Conflict: Quartz gives a range up to $549.99 and CNBC cites a $550 12 Pro — [Quartz](https://qz.com/amazon-alexa-tablets-fire-lineup-discontinued-100826); [CNBC](https://www.cnbc.com/2026/10/08/amazon-alexa-tablet-release.html).
- Amazon ended production of Fire tablets but still supports existing units. The previous base Fire HD 10 cost $155 — [TechRepublic](https://www.techrepublic.com/article/news-amazon-alexa-tablets-google-play-fire-tablets/); [Quartz](https://qz.com/amazon-alexa-tablets-fire-lineup-discontinued-100826).
- Earlier (August 2025) Reuters reporting described a single ~$400 Android tablet ("Kittyhawk") and said cheaper Fire tablets would stay on the Linux-based Vega OS — [9to5Google](https://9to5google.com/2025/08/20/amazon-fire-tablet-android-report/). The October 2026 launch superseded this: the whole line is now Android.
- Vega OS is already on the new Fire TV Stick 4K, and Vega devices do **not** support sideloading — [AFTVnews](https://www.aftvnews.com/these-are-the-fire-tvs-that-dont-support-sideloading-or-downloader-due-to-vega-os-replacing-fire-os/). A Vega tablet would therefore have been a dead end for Fully Kiosk.
- Amazon's help page says Show Mode is **not** compatible with the Fire HD 10 (13th Gen, 2023), Fire HD 8 (12th Gen) or Fire Max 11. Older Fire HD 10 generations do support it — [Amazon help: Switch to Show Mode](https://digprjsurvey.amazon.com/csad/help/node/G3JEMFDNH3FPB4XS). Flag: the page is undated and on an amazon.com subdomain.
- Fire OS ships its own embedded WebView, which cannot be updated separately and lags the Android System WebView. Home Assistant users report incompatibilities, such as camera codecs failing and "WebView render process unresponsive" in Fully, while Silk works — [HA Community: Fire HD 10 Kiosk Mode 2022](https://community.home-assistant.io/t/fire-hd-10-kiosk-mode-2022/491114); [HA Community: Fire HD vs Samsung](https://community.home-assistant.io/t/fire-hd-vs-samsung/586717). In one report, a Fully dashboard on WebView 70 went black — [SharpTools Community](https://community.sharptools.io/t/fire-tablet-fully-kiosk-no-longer-opening-sharptools-pages/16503).
- No source found any hub, dock or Show-Mode equivalent on the new Alexa Tablets — [TechCrunch](https://techcrunch.com/2026/10/08/amazon-unveils-new-alexa-tablets-with-alexa-and-google-play-store-access/) (absence of mention only).

#### Dedicated wall panels

- The Sonoff NSPanel Pro 120 was $109.90 on sale ($129.90 regular) in USD on Sonoff's store — [Sonoff central control panels](https://sonoff.tech/en-br/collections/central-control-panel). Flag: the USD price appeared on a Brazil storefront.
- The NSPanel Pro and several Tuya/Smatek panels run **Android 8.1**. Their stock WebView is too old for most dashboard apps, including Fully Kiosk, until it is updated over ADB — [blakadder: Update WebView on NSPanel Pro and Tuya panels](https://blakadder.com/android-panel-webview/).
- NSPanel Pro users report lag in state updates even after updating WebView, and performance varies between units — [HACF forum (French)](https://forum.hacf.fr/t/article-sonoff-nspanel-pro/30106?page=12). Fully needs about 3 s after wake to refresh switch states — [simon42 community (German)](https://community.simon42.com/t/empfehlungen-zur-integration-von-sonoff-nspanel-pro/42363). The panel has no restart button, so a hard reset means a power cycle — [SharpTools Community](https://community.sharptools.io/t/nspanel-sharptools/13409). The open-source "ha-paneld" service targets these weak in-wall Android panels alongside the HA Companion app — [HA Community: ha-paneld](https://community.home-assistant.io/t/ha-paneld-make-any-in-wall-android-panel-a-great-dashboard-for-home-assistant-free-open-source/1013158).

### Inferences

- With Fire tablets out of production, a "cheap Fire HD 10" is now only a used or clearance option. It also carries the outdated-WebView problem. The new Alexa Tablets are plain Android 16 with Play, so Fully Kiosk should install from Play, but they cost more than the Galaxy Tab A11+ and no source mentions a charge-limit setting.
- The 80% caps on the Samsung tablets and the iPad (A16) are the main defence against battery swelling (section 4). This argues against older iPads and against Lenovo's automatic-only 60% mode, which re-enables itself after updates.
- An OLED tablet (the Galaxy Tab S high-end line, the iPad Pro M4/M5) is the wrong panel type for a static dashboard. The LCD models (Tab A11+, Tab S10 FE, iPad A16) are better (section 4).

### Gaps

- No current Aqara panel (for example the Panel Hub S1 Plus or a 2026 Aqara display) price or Home Assistant suitability was found. Lenovo's 2026 tablet prices (Tab M11 / Tab Plus) were not found.
- No primary Apple page for the iPad (A16) price was fetchable (apple.com was not tried; support.apple.com was blocked).
- It is not known whether the Alexa Tablets have a charge limit, a hub/dock mode or an always-on mode.
- No Home Assistant "Voice PE"-style first-party wall panel exists in any source found. Voice PE is a speaker, not a display (from knowledge, unsourced).

## 2. Kiosk software: Fully Kiosk Browser, WallPanel, HA Companion, FreeKiosk, screen pinning; iPad options

### Takeaway

Fully Kiosk Browser (Android only) is still the de facto standard. A PLUS licence is a **one-time €8.90 + tax per device**, and it is required for motion detection and for the Home Assistant integration. WallPanel's maintained fork was **archived on 5 May 2025**. The open-source alternatives are FreeKiosk (MIT, MQTT plus HA auto-discovery), Kiosk Satellite (beta, non-commercial licence) and the HA Companion app used as a launcher. On iPad, the choices are Guided Access (free, ends on reboot), the HA iOS app's kiosk mode (reworked July 2026), Kiosk Pro ($24.99–$99.99 one-time), or supervised Single App Mode through Apple Configurator.

### Cited Findings

#### Fully Kiosk Browser

- The PLUS licence costs 8.90 € + tax per device. It is one-time and lifetime, and it can be moved to another device at any time. Volume licences are discounted for 10 or more devices. The optional Fully Cloud advanced features cost 1.18 €/month + tax per device, billed annually; basic cloud features are included with PLUS — [Fully PLUS Single License](https://license.fully-kiosk.com/license/single); [fully-kiosk.com](https://www.fully-kiosk.com/). Older third-party pages quote €7.90 as of October 2025 — [Maximum Meridian, 19 Oct 2025](https://maximum-meridian.com/smarthome/2025/10/19/fully-kiosk-browser) — or €5.90 — [SharpTools Community](https://community.sharptools.io/t/alternative-to-fully/2273). A $20–25 "no lifetime licence" claim comes from a low-quality blog and conflicts with the vendor page — [halloffame-online.com](https://halloffame-online.com/blog/touchscreen-kiosk-app-fully-locked-down-kiosk-browser-download).
- Motion features require PLUS — [HA Community: Fully Kiosk Browser thread](https://community.home-assistant.io/t/fully-kiosk-browser/41675). Fully can report motion from the camera or the gyroscope, but one user found it erratic — same thread.
- The REST API runs locally on port 2323, is password-protected and returns JSON. Commands include screen on/off, load the start URL or any URL, start/stop the screensaver, text-to-speech, bring the app to the foreground, and launch apps by package ID — [SharpTools Fully integration docs](https://docs.sharptools.io/bridge/integrations/fully-kiosk); [skill index summary](https://openskillindex.com/skills/majiayu000-claude-skill-registry-fully-kiosk) (third-party).
- Fully's own timer settings can switch the screen off at night and on in the morning. Users also drive screen on/off from Home Assistant through the REST API. Newer Fully versions report motion more reliably over MQTT. Users also report that camera motion may fail to wake the screen once the Android display timeout has fired — [HA Community: Fully thread, page 12](https://community.home-assistant.io/t/fully-kiosk-browser/41675?page=12).
- The Home Assistant core **Fully Kiosk Browser integration** requires a Fully PLUS licence, Remote Admin enabled, and the device IP and password. Its controls include motion detection on/off, screensaver on/off, the screensaver timer, and screen on/off. Its camera entity works only when motion detection is on — [home-assistant.io/integrations/fully_kiosk](https://www.home-assistant.io/integrations/fully_kiosk) (read through the [HA docs mirror](https://ha-doc.hasscn.top/integrations/fully_kiosk); the official page was blocked). It also exposes battery level and charging state, which can drive a smart-plug charge-limit automation — [Selora Homes integration overview](https://selorahomes.com/home-assistant/integrations/fully_kiosk/).
- Fully on Fire tablets inherits the outdated Fire OS WebView problems — [HA Community](https://community.home-assistant.io/t/fire-hd-10-kiosk-mode-2022/491114).

#### WallPanel (open source)

- The maintained fork TheTimeWalker/wallpanel-android took over the original ThanksMister project in 2022 and was **archived (read-only) on 5 May 2025**. The maintainer could not keep up with issues and is looking for someone to take over. The GitHub Releases section is empty. Features: MQTT and HTTP commands, camera motion and face detection to dismiss the screensaver, video streaming, QR codes. Minimum Android is 4.4 — [GitHub: TheTimeWalker/wallpanel-android](https://github.com/TheTimeWalker/wallpanel-android) (fetched).

#### Other Android options

- **HA Companion app (Android)**: it can be set as the device's default home app (launcher) to make a dedicated control panel. The docs recommend local push and keeping the screen on — [HA Companion docs: Android home app launcher](https://companion.home-assistant.io/docs/integrations/android-home-app-launcher). I found no 2026 dedicated kiosk mode in the Android app — [HA Community feature request](https://community.home-assistant.io/t/add-kiosk-mode-to-android-app/502618).
- **FreeKiosk**: MIT-licensed, Android 8.0+. It offers WebView, external app, dashboard and media modes; Device Owner lockdown; a REST API with 40+ endpoints; and MQTT with HA auto-discovery. True screen-off and reboot need Device Owner mode. The README shows v1.2.20, and a v2.0.0 beta adds MQTT camera snapshots and screenshots. The v2 notes warn that images are retained on the broker and that TLS is used only when the broker port is 8883 — [GitHub: RushB-fr/freekiosk](https://github.com/RushB-fr/freekiosk); [v2.0.0-beta.3 notes](https://newreleases.io/project/github/RushB-fr/freekiosk/release/v2.0.0-beta.3).
- **Kiosk Satellite**: first public beta (v0.1.0). A free Android kiosk browser for HA and companion to Voice Satellite. It needs Android 7.0+ and an HA long-lived access token, is licensed CC BY-NC-ND 4.0, and runs a remote admin UI on port 2324 — [release notes](https://newreleases.io/project/github/jxlarrea/kiosk-satellite/release/v0.1.0-beta); [GitHub discussion](https://github.com/jxlarrea/kiosk-satellite/discussions/1).
- **Webview Kiosk** (F-Droid) can wake the screen on MQTT commands — [F-Droid](https://f-droid.org/ko/packages/uk.nktnet.webviewkiosk/).
- **browser_mod** (HA custom integration) offers a motion binary sensor. In Fully it can react to physical motion through the device camera, and it can show camera popups — [hass-browser_mod docs (mirror)](https://docsearch.algolia.com/mcp/docs/repo/thomasloven/hass-browser_mod).

#### iPad options

- **HA iOS app kiosk mode**: reworked in 2026.7.0 (early July 2026). Version 2026.7.2 adds the commands `kiosk_set_screensaver_mode` and `kiosk_set_screensaver_brightness` — [Notebookcheck (German)](https://www.notebookcheck.com/Home-Assistant-Neues-Update-erweitert-Kiosk-Modus.1354575.0.html).
- **Kiosk Pro** (Kiosk Group) is a one-time licence per device. Lite is free. Basic is $24.99 (offline/local mode, remote settings, no watermark). Plus is $49.99 (screensaver loops, **night mode**, managed app config through MDM). Enterprise is $99.99 (barcode, printer, Zapier). The Remote Management Server costs $149/year per device — [Kiosk Pro pricing](https://kioskgroup.com/pages/pricing).
- **Guided Access** is free. It has its own Display Auto-Lock and ends on reboot. **Single App Mode** needs Apple Configurator or MDM — sources as in section 1 ([Kiosk Group](https://support.kioskgroup.com/article/1024-enable-auto-lock), [Virtual Front Desk](https://www.virtualfrontdesk.com/knowledge-base-article/ipad-kiosk-mode-guided-access)).

### Inferences

- For Android, Fully PLUS (about $10 one-time) is cheap enough that it is the default. FreeKiosk is the best fully open-source fallback, while WallPanel is effectively dead. The HA integration makes screen, brightness, motion and battery controllable from automations, which is what the night-mode and charge-limit recipes need.
- On iPad, nothing equals Fully's motion-wake and REST control. The HA iOS kiosk mode (if the dashboard runs inside HA) or Kiosk Pro Plus is the closest. A custom (non-HA) web page on iPad gets Guided Access plus a Home Screen web app, with no remote screen control.
- On Android, set Fully (or FreeKiosk) as the launcher or Device Owner so it survives reboots after OS updates. A Guided Access session does not survive a reboot.

### Gaps

- The exact list of Fully features that are PLUS-only (versus free) could not be read from fully-kiosk.com (blocked). The latest Fully version and its minimum Android version were also not confirmed.
- Android screen pinning (App pinning) was not sourced. It is a built-in, free, basic lock that does not survive reboot (from general knowledge, unverified here).
- Kiosk Pro's current iPadOS 26 compatibility was not checked.

## 3. Web platform features the page can rely on (Safari/WebKit on iPadOS vs Chrome/Android WebView)

### Takeaway

On current versions, both engines support what a dashboard needs. The Screen Wake Lock API works in Safari 16.4+, and inside Home Screen web apps since iOS 18.4 (it was broken there before). On Android, Chrome and the WebView support it too. The two traps are (a) **HTTPS on the LAN**: Safari needs a properly trusted CA certificate profile for HTTPS and WebSockets to a self-signed server, and (b) Chrome's **Local Network Access** permission, enforced since Chrome 142 (28 October 2025), for public-origin pages that call private IPs. Android WebView (inside Fully) is exempt from the LNA prompt. Autoplay with sound always needs a user gesture.

### Cited Findings

- **Screen Wake Lock**: supported in Safari since 16.4, and in Chrome — [Chrome for Developers: Screen Wake Lock](https://developer.chrome.com/docs/capabilities/web-apis/wake-lock). WebKit bug 254545 ("Wake Lock API works in Safari but not when added to Home Screen") is RESOLVED FIXED — [WebKit Bugzilla 254545](https://bugs.webkit.org/show_bug.cgi?id=254545). The fix shipped in **iOS 18.4** — [Progressier PWA capability page](https://progressier.com/pwa-capabilities/screen-wake-lock). Conflict: MDN-derived tables still carry a "does not work in standalone Home Screen web apps" note, which looks stale — [w3cub mirror of MDN/caniuse table](https://docs.w3cub.com/browser_support_tables/wake-lock). The lock can be released when the tab is hidden, on low battery or by OS power policy, so the page must re-request it on `visibilitychange` — [Vaadin docs](https://origin.vaadin.com/docs/next/flow/browser-apis/wake-lock-api). Wrap the request in try/catch — [Progressier](https://progressier.com/pwa-capabilities/screen-wake-lock).
- A similar standalone-mode bug is still open for Picture-in-Picture: `requestPictureInPicture()` fails in Home Screen web apps (filed December 2025) — [WebKit Bugzilla 303885](https://bugs.webkit.org/show_bug.cgi?id=303885).
- **Web App Manifest / install**: in iPadOS 26, Add to Home Screen opens sites as web apps by default, and sites that have a manifest still get its settings — [WebKit WWDC25 post](https://webkit.org/?p=16993).
- **Autoplay (WebKit)**: video that is `muted`, or has no audio track, may autoplay without a gesture. If it gains audio or is unmuted without a gesture, it pauses. Autoplay video pauses when it is off-screen. A `click` handler calling `play()` counts as a gesture; a `canplaythrough` handler does not. Use `playsinline` for inline playback — [WebKit: New video policies for iOS](https://webkit.org/blog/6784/new-video-policies-for-ios/) (2016, still the base policy). In Low Power Mode, playback needs a user interaction even when muted — [nanocosmos docs](https://docs.nanocosmos.de/docs/nanoplayer/nanoplayer_autoplay).
- **Self-signed certificates on iPad Safari**: install the certificate as a profile, then enable it under Settings > General > About > Certificate Trust Settings. The certificate must have `basicConstraints=CA:true` and a SAN matching the hostname. Use a hostname (for example `.local` or a DNS name), not a bare IP. Without this, Safari will not allow WebSocket connections — [code-server iPad docs (v3.12)](https://coder.com/docs/code-server/v3.12.0/ipad). Flag: that doc dates from around 2021; the trust-settings path is also documented for iOS 17 — [Cybozu iOS 17 certificate guide](https://jp.cybozu.help/en/settings/browser/certificate/iphone/ios17.html).
- **Chrome Local Network Access (LNA)**: a permission prompt (announced 9 June 2025) for public sites making requests to private-network devices, covering `fetch()`, subresources and subframes. It is secure-context only. Chrome **exempts these requests from mixed-content blocking** when it knows before DNS resolution that the target is local: a private IP literal, a `.local` name, or `fetch(..., {targetAddressSpace: "local"})`. A public hostname that resolves to a private IP gets no exemption — [Chrome for Developers: Local Network Access](https://developer.chrome.com/blog/local-network-access). It has been enforced since **Chrome 142 (28 October 2025)**. Since Chrome 145 the permission is split into `loopback-network` and `local-network` — [Dynamsoft FAQ on Chromium 142 LNA](https://www.dynamsoft.com/web-twain/docs/faq/chromium-142-local-network-access-issue.html).
- **Android WebView and LNA**: the restrictions do not apply to WebView; LNA is "unconditionally granted" there. Android is adding a separate app-level Local Network permission for the app that embeds the WebView — [blink-dev Intent to Ship: Local network access restrictions](https://groups.google.com/a/chromium.org/g/blink-dev/c/cwu_RUmBpzY/m/JWenvGViCAAJ). Flag: this is from the 2025 design thread.
- **WebView version** matters for Fully and the Companion app. On Fire OS it is frozen to the OS — [HA Community](https://community.home-assistant.io/t/fire-hd-10-kiosk-mode-2022/491114). On Android 8.1 panels it must be updated over ADB — [blakadder](https://blakadder.com/android-panel-webview/).

### Inferences

- The simplest robust design serves the dashboard and its API from the **same origin** on the home server (for example `https://dashboard.home.arpa` or HA itself) and has the server make all calls to HTTP devices. This avoids mixed-content and LNA problems entirely, and keeps device credentials off the tablet (section 6).
- If the page uses HTTPS on the LAN, use a real certificate for a domain you own (Let's Encrypt DNS-01 with split-horizon DNS, or Tailscale's MagicDNS/Serve certificate) rather than a self-signed one. Otherwise the iPad needs a manually trusted root profile, and service workers/PWA install require a trusted secure context. That last point is inference from the secure-context rules, not verified in this session.
- WebSockets, Server-Sent Events, `prefers-color-scheme`, service workers and Pointer/Touch events are baseline features in current Safari (iPadOS 18/26) and Chromium WebView. I expect no per-platform shims to be needed beyond wake-lock re-acquisition and gesture-gated audio. This is unverified here because caniuse/MDN were blocked.
- Sound (Sonos/Alexa/TTS) should be played by the speakers, not the page, because page audio needs a gesture on both engines.

### Gaps

- caniuse.com and MDN tables for SSE, WebSockets, service workers, `prefers-color-scheme` and display-mode `fullscreen` vs `standalone` on iPadOS 26 could not be fetched. I did not confirm whether iPadOS 26 honours `display: fullscreen` (historically it falls back to `standalone`).
- Service-worker behaviour with a self-signed (manually trusted) certificate on iPadOS was not found.
- How iOS Safari handles `http://` device calls from an `https://` page (mixed content is generally blocked, with no LNA-style exemption) was not sourced.

## 4. Always-on hardware pitfalls: battery swelling, burn-in, heat, Wi-Fi sleep, auto-updates, power delivery, mounts

### Takeaway

Battery swelling from constant charging is the main long-term failure. Community practice is a built-in charge cap (Samsung 80/85%, iPad 80%) or Home Assistant smart-plug cycling (for example on at 15–40%, off at 80–85%), driven by the battery sensor that Fully reports. Use an LCD rather than OLED, keep the tablet away from heat, and plan for updates that reboot the device or reset settings. PoE-to-USB-C in-wall adapters with USB-PD are the cleanest way to power a flush mount.

### Cited Findings

- **Swelling and mitigation (community)**: the common view is that a smart plug should cut power so the battery does not swell — [HA Community: How best to power a tablet?](https://community.home-assistant.io/t/how-best-to-power-a-tablet/686488). One user stops charging at 85% and restarts below 15% from a battery reading sent to openHAB — [openHAB Community: wall-mounted tablet setups](https://community.openhab.org/t/your-setup-for-a-wall-mounted-tablet/22540?page=7). Others keep the tablet between about 40% and 80%, mention Lenovo/Samsung built-in caps, and argue that limiting charge only delays failure. Swollen batteries have popped screens off the frame — [HA Community: Tablet as wall panel](https://community.home-assistant.io/t/tablet-as-wall-panel/734479); [SmartThings Community: plugged in 24/7?](https://community.smartthings.com/t/do-you-leave-your-smart-home-tablet-plugged-in-24-7/246866); [HomeSeer forum: wall tablet power solution](https://forums.homeseer.com/forum/homeseer-products-services/general-discussion-area/1433414-wall-tablet-power-solution-needed?p=1433475). These are forum anecdotes, not safety guidance.
- **Built-in caps**: Samsung Maximum (80%) / Basic / Adaptive on One UI 6.1+, or a fixed 85% earlier — [Samsung Support](https://www.samsung.com/au/support/mobile-devices/galaxy-battery-protection-feature-in-one-ui-6-1/). iPad 80% (with an occasional 100% recalibration) on A16, M2 Air, M4 Pro and A17 Pro mini — [Apple Support 118418](https://support.apple.com/en-gb/118418). Lenovo automatic 60% after 7 days — [Lenovo/Bluestar](https://reach.bluestarinc.com/eu-lenovo-launch-tablets).
- **Burn-in**: mainly an OLED concern. Keep brightness moderate and avoid static white elements for months. **Heat** is what most shortens a permanently plugged-in tablet's life, so avoid direct sun and walls above heater vents — [AndroidGuys: spare tablet as wall dashboard](https://androidguys.com/promoted-news/how-to-turn-a-spare-android-tablet-into-a-wall-dashboard-for-your-home/). Recent tablet/phone burn-in reports are rare, but OLED degrades over time — [MyNextTablet: OLED burn-in](https://mynexttablet.com/oled-burn-in/). Static dashboard elements (tiles, status strips, clocks) are the classic burn-in risk; mitigations are dimming, auto-hiding and sleeping — [KTC: OLED burn-in risk for office dashboards](https://us.ktcplay.com/blogs/technology-hub/oled-burn-in-office-dashboard) (vendor with a Mini-LED bias).
- **Auto-updates and reboots**: a reboot ends Guided Access on iPad — [Virtual Front Desk](https://www.virtualfrontdesk.com/knowledge-base-article/ipad-kiosk-mode-guided-access). A Lenovo update re-enabled Battery Protection Mode on kiosk tablets — [Hexnode forum, Jan 2026](https://www.hexnode.com/forums/topic/how-to-disable-battery-protection-mode-on-lenovo-tablets/). Fully updates have broken Fire tablet dashboards; one suggested fix was asking Fully for an older APK — [HA Community: Fully thread](https://community.home-assistant.io/t/fully-kiosk-browser/41675?page=27).
- **Power over PoE**: POE Texas sells PoE+ (802.3at) to USB-C splitters that negotiate USB-PD profiles to keep a tablet charged, with power-only USB-C and separate gigabit data — [POE Texas GAT-USBC](https://reach.dog/shop/poe-texas/products/gat-usbc). There are also in-wall junction-box versions (about 25 W, a USB-C power+data output plus a power-only output) — [PB Tech: POE Texas PT-AT-HUB-USBC](https://www.pbtech.com/product/TAAPOE1021/POE-Texas-PT-AT-HUB-USBC-PoE-to-USB-C-Power--Data); flag: exact output specs to verify. Some flush units fit standard 86 mm wall boxes — [Newegg listing](https://www.newegg.com/p/3C6-01ZY-02605). 5 V-only splitters (UCTRONICS U6115, SMLIGHT) suit a Raspberry Pi but may charge large tablets slowly or not at all — [desertcart U6115](https://www.desertcart.sg/cmn/products/211481483); [SMLIGHT PoE splitter](https://www.skroutz.gr/s/68467602/smlight-poe-adapter-splitter.html). Users discuss charging tablets over Ethernet runs — [SmartThings Community](https://community.smartthings.com/t/correct-cable-for-charing-tablet-via-ethernet/226926).
- **Battery-less conversion**: one commenter argues for removing the battery altogether, since a cap only delays failure — [HA Community: Tablet as wall panel](https://community.home-assistant.io/t/tablet-as-wall-panel/734479). No step-by-step guide was verified.

### Inferences

- Prefer a tablet with a **user-set** 80% cap (Galaxy Tab A11+/S10 FE on One UI 6.1+, iPad A16) **plus** a smart plug as a second layer, or a smart plug alone for older tablets. Leave a few centimetres of air gap behind the mount and check the tablet once a year for case bulging.
- A PoE-to-USB-C in-wall box needs an 802.3at switch or injector. Confirm the adapter does USB-PD at the tablet's charging voltage, because a 5 V-only splitter may not keep up with a 10–11" tablet while the screen is on.
- Updates matter in both directions: they bring security fixes but can reset kiosk settings. On Android, put Fully/FreeKiosk as launcher or Device Owner. On iPad, use supervised Single App Mode if surviving reboots matters.

### Gaps

- Wi-Fi sleep settings ("keep Wi-Fi on during sleep", Samsung "Put unused apps to sleep" / battery optimization exemptions for Fully) were not sourced.
- Specific wall-mount products (VidaMount, flush in-wall frames, 3D-printed mounts, magnetic mounts) and their US prices were not researched.
- No engineering source quantifies swelling risk at a 100% float charge versus an 80% cap on tablets.
- There is no verified battery-removal or "power-only" conversion guide for current tablets, and the safety implications are unknown.

## 5. Night mode, dimming and presence; privacy of an always-on camera

### Takeaway

Fully PLUS can wake the screen from camera motion and lets HA dim or switch the screen. Users find camera motion "touchy", since it triggers on light changes, and some report it fails after the Android display timeout. Many therefore prefer an **external PIR/mmWave sensor plus an HA blueprint** to drive the screen through the Fully integration, which also avoids an always-on camera. On iPad, the HA iOS kiosk mode (2026.7.2) adds a screensaver and brightness commands, and Kiosk Pro Plus has a night mode.

### Cited Findings

- Fully visual motion detection plus "exit screensaver on motion" wakes the screen when someone passes. A black-wallpaper screensaver hides the dashboard until motion (a PLUS feature) — [Yoav Mor: wall-mounted tablet dashboard](https://smarthome.yoavmor.com/home-assistant/controlling-your-home-assistant-dashboard-with-a-wall-mounted-tablet/).
- Fully's built-in motion detection was "too touchy and got triggered by changing light levels all the time". One user dims and restores brightness through Fully remote control from an external motion sensor next to the tablet — [HA Community: HA on my wall](https://community.home-assistant.io/t/ha-on-my-wall/145998).
- A community blueprint turns the screen on immediately on motion and off when the sensor clears, instead of a blind countdown — [HA Community blueprint: Intelligent Wall Tablet Screen Control](https://community.home-assistant.io/t/blueprint-intelligent-wall-tablet-screen-control-on-off-on-motion/1016447). A free alternative turns an Android device's camera (IP Webcam) into an HA motion sensor — [HA Community](https://community.home-assistant.io/t/simple-and-free-turn-an-android-tablet-phone-on-off-based-on-camera-motion-detection/836908).
- Camera motion may not wake the screen after the Android display timeout. Users schedule screen off at midnight and on at 06:00 in Fully, or drive it from HA — [HA Community: Fully thread, page 12](https://community.home-assistant.io/t/fully-kiosk-browser/41675?page=12).
- HA's Fully integration exposes screen on/off, brightness, screensaver and motion-detection toggles. Its camera entity works only when motion detection is on — [HA docs mirror](https://ha-doc.hasscn.top/integrations/fully_kiosk).
- HA iOS app 2026.7.2 adds `kiosk_set_screensaver_mode` and `kiosk_set_screensaver_brightness` — [Notebookcheck](https://www.notebookcheck.com/Home-Assistant-Neues-Update-erweitert-Kiosk-Modus.1354575.0.html). Kiosk Pro Plus ($49.99) includes night mode — [Kiosk Pro pricing](https://kioskgroup.com/pages/pricing).
- **Privacy**: FreeKiosk v2 beta publishes camera snapshots and screenshots over MQTT. They are retained on the broker, and TLS is used only on port 8883 — [FreeKiosk v2.0.0-beta.3 notes](https://newreleases.io/project/github/RushB-fr/freekiosk/release/v2.0.0-beta.3). Fully's camera entity makes the tablet camera viewable in HA — [HA docs mirror](https://ha-doc.hasscn.top/integrations/fully_kiosk).

### Inferences

- For a hobbyist, the most reliable and privacy-friendly presence source is an external mmWave/PIR sensor already in HA. It drives the screen and brightness through the Fully integration, with Fully's camera motion detection off, and a schedule as a backstop. If camera motion is used, keep the HA camera entity disabled, or restrict it to admins, so the tablet does not become an indoor camera feed. If MQTT carries images, use TLS and don't retain them.
- Ambient-light dimming: Fully exposes the light sensor to HA as a sensor (inferred from the "device sensors" description; not verified). An HA automation can map lux to brightness, plus dark mode at night through `prefers-color-scheme` or a page theme toggle.

### Gaps

- It is not confirmed whether Fully processes camera frames only on the device, or whether there is any camera-in-use indicator on Android 12+ tablets during Fully motion detection (Android shows a privacy indicator; behaviour with Fully was not verified).
- No source on proximity-sensor use (most tablets lack one) or Fully's light-sensor entity was fetched.

## 6. Security of a wall tablet that holds tokens

### Takeaway

Anyone with physical access to a wall tablet gets whatever its stored credential can do. A Home Assistant long-lived token carries **the full permissions of its user**, and Fully/Kiosk Satellite expose remote-admin ports on the LAN. Keep secrets on the server: the page should call a same-origin backend that holds device/API tokens and offers only the actions the dashboard needs. Use a dedicated non-admin HA user (or a scoped proxy), set an admin password on the kiosk app, and use local-only access, with Tailscale (no port forwarding) for anything remote.

### Cited Findings

- An HA long-lived access token has the same permissions as the user account it was created under. Use separate tokens per integration and revoke exposed tokens from the Security tab — [CodeWords: Home Assistant access tokens (May 2026)](https://www.codewords.ai/blog/home-assistant-access-token).
- Historically, non-admin HA users could not create long-lived tokens because advanced mode was admin-only, so an admin had to generate them — [HA frontend discussion #11171](https://github.com/home-assistant/frontend/discussions/11171). Flag: this is older and may have changed.
- Kiosk Satellite requires an HA long-lived token and runs a remote admin UI on port 2324 that mirrors every setting. That makes it an exposure point on the LAN — [Kiosk Satellite release](https://newreleases.io/project/github/jxlarrea/kiosk-satellite/release/v0.1.0-beta). Fully's HA integration requires Remote Admin with a password, and the REST API listens on port 2323 — [HA docs mirror](https://ha-doc.hasscn.top/integrations/fully_kiosk); [SharpTools docs](https://docs.sharptools.io/bridge/integrations/fully-kiosk).
- A community "Gatekeeper" app lets guests trigger chosen actions without HA access. It warns not to use it for door unlocks or other sensitive actions — [HA Community: Gatekeeper](https://community.home-assistant.io/t/gatekeeper-allow-anyone-to-trigger-actions-without-accessing-home-assistant/1019408).
- **Remote access with Tailscale**: install the Tailscale app in HA (Settings > Apps). HA is then reachable at its 100.x IP on port 8123 or by MagicDNS name, with no port forwarding (it works behind CGNAT). Use Funnel only when public exposure is intended. Tailscale Serve can provide a private HTTPS name — [Tailscale blog, 16 July 2026](https://www.tailscale.com/blog/remotely-access-home-assistant); [NATChecker guide](https://natchecker.com/blog/tailscale-home-assistant).

### Inferences

- What an attacker with the tablet could reach: everything the stored token allows (all HA entities for an admin token), the Fully remote-admin/REST API (screen, URL, camera if enabled), the Wi-Fi network itself (the PSK is on the device), and any API tokens embedded in page JavaScript (Sonos, Google, Nest). Mitigations: a dedicated low-privilege user and token per tablet, revoked if the tablet goes missing; server-side storage of Google/Nest/Sonos credentials; a page that talks only to its own backend; an IoT VLAN or firewall rule so the tablet reaches only the dashboard server; Fully remote-admin password set, with remote admin reachable only from the server; and no unlock or alarm-disarm actions without a PIN step on the server.
- Keep the wall tablet itself **local-only**: it does not need Tailscale or Nabu Casa. Remote access belongs on the owner's phone.

### Gaps

- Nabu Casa remote UI behaviour and security for kiosk devices was not researched.
- Whether current HA (2026.x) lets non-admin users create long-lived tokens, and whether per-entity permissions exist, was not verified (HA has historically lacked fine-grained ACLs; unverified here).
- iPad-specific exposure (Safari stored credentials, Guided Access passcode strength) was not researched.

## 7. Recommendation for a hobbyist (synthesis)

### Takeaway

The best default for an owner building their own page is a **Samsung Galaxy Tab A11+ (about $160–250) or Tab S10 FE (about $335–450)**. Configure Battery protection "Maximum" (80%), run **Fully Kiosk Browser PLUS (€8.90 one-time)** as launcher, add the HA Fully integration for screen, brightness and motion control, use an external presence sensor, and power it from an in-wall PoE-to-USB-C (USB-PD) adapter or a hidden outlet with a smart plug as a backup. Choose an **iPad (A16), $349**, only if the owner values WebKit polish and the 80% cap over remote screen control. Avoid buying Fire tablets now, and choose an NSPanel-class panel only for a small flush-mounted controller where the slow WebView is acceptable.

### Cited Findings

- Fully PLUS price and licensing — [Fully PLUS Single License](https://license.fully-kiosk.com/license/single).
- Galaxy Tab A11+ / S10 FE prices — [9to5Toys](https://9to5toys.com/2026/06/24/2026-galaxy-tab-prime-day/); [Productchart A11+](https://www.productchart.com/tablets/similar_to_50084); [Productchart S10 FE](https://www.productchart.com/tablets/similar_to_50991).
- Samsung 80% Maximum mode — [Samsung Support](https://www.samsung.com/au/support/mobile-devices/galaxy-battery-protection-feature-in-one-ui-6-1/).
- iPad (A16) 80% limit — [Apple Support 118418](https://support.apple.com/en-gb/118418).
- Fire tablets discontinued; Alexa Tablets from $229.99 — [TechCrunch](https://techcrunch.com/2026/10/08/amazon-unveils-new-alexa-tablets-with-alexa-and-google-play-store-access/); [Quartz](https://qz.com/amazon-alexa-tablets-fire-lineup-discontinued-100826).
- NSPanel Pro on Android 8.1 with an old WebView — [blakadder](https://blakadder.com/android-panel-webview/).
- Pixel Tablet discontinued — [Tech Advisor](https://www.techadvisor.com/article/3232946/farewell-pixel-tablet-i-barely-knew-you.html).

### Inferences

- Comparison summary, synthesized from the sources above:
  - **Galaxy Tab A11+/S10 FE + Fully**: user-set 80% cap; full kiosk, REST and MQTT; HA integration; Play-updated Chromium WebView (fresh engine); LCD. Cons: Samsung bloat, optimization settings to tame. Best overall.
  - **iPad (A16)**: 80% cap; excellent Safari; iPadOS 26 web-app default; wake lock works in Home Screen apps since 18.4; long OS support. Cons: no Fully equivalent; Guided Access ends on reboot; no remote screen-off for a non-HA page; manual certificate trust for self-signed HTTPS; $349.
  - **Alexa Tablet 8/11 (Android 16)**: should run Fully from Play and gets four years of updates. Cons: unknown charge limit; Amazon launcher; $229.99+, so no cheaper than Samsung. Too new to have community kiosk reports.
  - **Used Fire HD 10 / Max 11**: cheap. Cons: frozen Fire OS WebView, no Play, discontinued line, Show Mode absent on recent generations. Not recommended for new purchases.
  - **NSPanel Pro-class in-wall Android panel**: about $110–130, flush-mounted, mains-powered (no battery to swell, inferred from the panel design; verify per model). Cons: Android 8.1, needs an ADB WebView update, slow, small screen. Good as a secondary room controller, not as the main dashboard.
- Build the page to tolerate any of these hosts: a same-origin backend; wake-lock with re-acquire on `visibilitychange`; dark theme at night; no autoplay with sound; reconnecting WebSocket/SSE; large touch targets. Then the tablet choice stays reversible.

### Gaps

- No hands-on long-term reports yet for the October 2026 Alexa Tablets as kiosks.
- US availability and price of the Galaxy Tab A11 (non-plus) and Lenovo 2026 models were not confirmed.
