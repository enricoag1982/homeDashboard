---
status: concluded
date: 2026-10-17
authors: [Claude Code]
related: [2026-10-17-tablet-home-dashboard-options]
---

# Google data (Calendar, Keep and alternatives, Gmail) and weather for a self-built home dashboard (US, personal @gmail.com account, as of October 2026)

Method note for the report writer: direct page fetching was blocked in this research session (the egress proxy refused connections to developers.google.com, support.google.com, weather.gov and others). Every finding below therefore comes from search-engine extracts of the cited pages, not from a full read of each page. Primary-source URLs are given wherever the extract came from them. Where only secondary sources were available, or sources disagreed, this is flagged inline. Dates in brackets are publication or "as seen" dates where the search result gave one.

## 1. Google Calendar API for a personal project: Cloud project, consent screen, scopes, verification, and the 7-day refresh-token expiry

### Takeaway

A personal dashboard can use the official Calendar API for free. It needs a Google Cloud project with an "External" OAuth app; a consumer account has no "Internal" option. Read-only calendar scopes are treated as sensitive. The working pattern for hobbyists is to switch the OAuth app from "Testing" to "In production" and *not* submit it for verification. Google exempts personal use (under 100 users) from verification. The cost is an "unverified app" warning at consent and a lifetime 100-user cap. In exchange, refresh tokens no longer expire after 7 days. New for 2026: Calendar API quotas changed for projects created on or after May 1, 2026, and Google plans to bill usage above quota later in 2026.

### Cited Findings

- **Testing status means 7-day authorizations.** In "Testing" publishing status a project is limited to 100 listed test users, and "Authorizations by a test user will expire seven days from the time of consent." — [Google Cloud Help: When is verification not needed](https://support.google.com/cloud/answer/13464323?hl=en); [Google Cloud Help: Manage App Audience](https://support.google.com/cloud/answer/15549945?hl=en)
- **Production stops the 7-day expiry.** Once the app is "In production", refresh tokens generally do not expire unless they are revoked or go unused for a long time (about six months). — search summary of Google OAuth docs; secondary write-ups: [unified.to](https://unified.to/blog/why_google_oauth_returns), [sailjada (May 2026)](https://tech.sailjada.com/posts/2026-05-06-1856.html)
- **Re-consent after publishing.** Tokens issued while the app was in Testing may keep their 7-day expiry. Community advice is to re-consent after publishing so a new token is issued. — [sailjada (May 2026)](https://tech.sailjada.com/posts/2026-05-06-1924.html)
- **Other ways a refresh token dies.** There is a limit of 100 live refresh tokens per Google Account per OAuth client ID; when it is exceeded, the oldest token is silently invalidated. Revocation and inactivity also invalidate tokens. — [unified.to](https://unified.to/blog/why_google_oauth_returns) (secondary, summarizing Google's OAuth docs)
- **Personal-use exemption from verification.** Google's help says that if the app is "for your personal use (fewer than 100 users)", you and your users can keep using it without verification, though users must click through the "unverified app" warnings. — [Google Cloud Help: When is verification not needed](https://support.google.com/cloud/answer/13464323?hl=en)
- **Google's definition of personal use.** The OAuth policies define it as an app "not shared with anyone else or will be used by fewer than 100 people (all of whom are known personally to you)." — [Google OAuth 2.0 Policies](https://developers.google.com/identity/protocols/oauth2/policies)
- **The 100-user cap is for life.** The unverified-app user cap "applies over the entire lifetime of the project, and it cannot be reset or changed." The unverified-app screen and cap apply when the app requests sensitive or restricted scopes before verification. — [Google Cloud Help: Unverified apps](https://support.google.com/cloud/answer/7454865?hl=en); [OAuth app state overview](https://developers.google.com/identity/protocols/oauth2/production-readiness/overview)
- **Calendar scopes.** `calendar.readonly` gives read-only access to calendars. `calendar.events.readonly` lets the app "view events on all your calendars". Google advises choosing "the most narrowly focused scope possible". Events: list accepts `calendar.readonly`, `calendar`, `calendar.events.readonly` and others. — [Choose Google Calendar API scopes](https://developers.google.com/workspace/calendar/api/auth); [Events: list](https://developers.google.com/workspace/calendar/api/v3/reference/events/list)
- **Granular scopes.** Google also lists:
  - `calendar.events.owned`: see, create, change and delete events on calendars you own.
  - `calendar.calendarlist.readonly`: see the list of calendars you are subscribed to.
  - `calendar.app.created`: create secondary calendars and manage events only on those.
  — [OAuth 2.0 Scopes for Google APIs](https://developers.google.com/identity/protocols/oauth2/scopes)
- **How sensitive the calendar scopes are (not confirmed from Google's own table).**
  - A third-party guide lists `calendar.events.readonly` as **Sensitive**: verification is needed but no security assessment. — [Nylas: verification guide](https://developer.nylas.com/docs/provider-guides/google/google-verification-security-assessment-guide/); [Recall.ai](https://docs.recall.ai/docs/calendar-v2-google-calendar)
  - Nylas pages disagree on `calendar.readonly`: one calls it non-sensitive (metadata only), another sensitive. — [Nylas OAuth scopes](https://developer.nylas.com/docs/cookbook/use-cases/build/oauth-scopes-email-calendar/)
  - Nylas also lists `calendar.events` as "restricted". That **contradicts** Google's restricted-scope verification page, which describes restricted scopes as Gmail and Drive scopes. — [Google: Restricted scope verification](https://developers.google.com/identity/protocols/oauth2/production-readiness/restricted-scope-verification)
- **Where to check a scope's tier.** Sensitive scopes "require review by Google and have a sensitive indicator on the Google Cloud Console's OAuth consent screen configuration page". The console shows each scope's tier when you add it. — [OAuth 2.0 Scopes for Google APIs](https://developers.google.com/identity/protocols/oauth2/scopes)
- **Calendar API quotas and billing in 2026.**
  - Limits: 10,000 requests per minute per project and 600 requests per minute per user per project, on a sliding window. Exceeding them returns 403 or 429 with a `usageLimits` reason.
  - Cost: "All standard use of the Google Calendar API is available at no additional cost. Exceeding the quota request limits is planned to incur charges to your Google Cloud billing account later in 2026."
  - Which projects get the new quotas: projects created on or after May 1, 2026. Projects that used the API between November 2025 and April 2026 keep their previous quotas.
  - Google says it will give at least 90 days' notice before billing changes.
  — [Calendar API usage limits](https://developers.google.com/workspace/calendar/api/guides/quota); [Calendar API release notes](https://developers.google.com/workspace/calendar/release-notes)
- **Older quota figure (conflict).** An older Google page gives the Calendar API quota as 1,000,000 requests per day per project. This conflicts with the per-minute figures above, which apply to new projects. — [Apps Script quotas page (summary)](https://developers.google.com/apps-script/guides/services/quotas)
- **Inactive OAuth clients are deleted.** Since mid-2025, "OAuth clients inactive for 6 months will be automatically deleted". Owners are notified, and a deleted client can be restored within 30 days. — [Google Developers Blog: Usability and safety updates to Google Auth Platform](https://developers.googleblog.com/en/usability-and-safety-updates-to-google-auth-platform/); some notices were sent in error: [Google dev forum](https://discuss.google.dev/t/automatic-deletion-of-unused-oauth-clients/190380)
- **Home Assistant follows the same setup.** Its Google Calendar docs say to publish the OAuth app ("Under Publishing status, select Publish app. Otherwise, your credentials will expire every 7 days") and to use Web Auth credentials with the my.home-assistant.io redirect. — [Home Assistant: Google Calendar](https://www.home-assistant.io/integrations/google/)
- **Warning remains after publishing.** Production removes the 7-day expiry but not the unverified-app warning. One user saw a "verification required because of sensitive scopes" prompt when publishing. — [HA Community: Google Assistant SDK weekly re-auth](https://community.home-assistant.io/t/google-assistant-sdk-weekly-re-authorization-of-account/535887); [scoutjobs issue](https://github.com/ZoolCoder/scoutjobs/issues/41)
- **Apps Script quotas for consumer accounts.** For gmail.com accounts Apps Script allows 5,000 calendar events created per day, 90 minutes of total trigger runtime per day, and 6 minutes per execution. `doGet`/`doPost` web-app handlers are not subject to the 30-second simple-trigger cap. — [Apps Script quotas](https://developers.google.com/apps-script/guides/services/quotas); [Simple triggers](https://developers.google.com/apps-script/guides/triggers)

### Inferences

- Recommended setup for one household:
  1. Create a Cloud project with an External user type.
  2. Request only `calendar.readonly`, or `calendar.events.readonly` plus `calendar.calendarlist.readonly`. Add `calendar.events` / `calendar.events.owned` only if the tablet must create events.
  3. Publish to "In production" without submitting for verification.
  4. Accept the unverified-app screen once ("Advanced > Go to app").
  5. Store the refresh token server-side, on the home server or in Home Assistant, never on the tablet.
- One OAuth client can serve Calendar, Tasks and Gmail (with gmail.labels) for the whole household. Each family member who consents counts toward the lifetime 100-user cap. A household will never reach it.
- Known pitfalls to design around:
  - Tokens minted while the app was in Testing: re-consent after publishing.
  - The 6-month inactive-client deletion: keep the dashboard polling, or note the 30-day restore window.
  - A password change or revocation from the Google Account security page invalidates tokens.
  - Planned over-quota billing is irrelevant at dashboard polling rates. One poll per minute is far under 600 per minute per user. Even so, attaching a billing account is no longer cost-free insurance. Set a budget alert if billing is enabled.
- Alternative not fully researched: an Apps Script web app that runs as the owner and returns calendar or tasks JSON avoids running an OAuth client on the home server. Its URL becomes a bearer secret, much like an ICS link. This is an inference; no source evaluated it for dashboards.

### Gaps

- Google's own Calendar scope table, with its Sensitive / Non-sensitive column, could not be read directly. The tier of `calendar.readonly`, `calendar.calendarlist.readonly`, `calendar.events.owned` and `calendar.app.created` is unconfirmed. Check it in the Cloud Console "Data access" page when adding scopes.
- Google's exact refresh-token inactivity window (commonly "six months") was confirmed only via secondary sources.
- Google had not yet published the final 2026 Calendar API billing start date or prices.

## 2. Simpler alternative: the private "Secret address in iCal format" (ICS), parsing libraries, and Home Assistant's Google Calendar and Remote Calendar integrations

### Takeaway

The secret iCal address needs no OAuth at all. It is a bearer URL per calendar, found under Settings → (calendar) → Integrate calendar. It is the simplest read-only path. The costs are privacy, since anyone with the URL can read the calendar, and freshness, since Google publishes no update SLA and anecdotal lag runs from minutes to hours. Home Assistant offers both the OAuth-based Google Calendar integration (read or read-write, polled) and a core "Remote Calendar" integration for ICS URLs. Remote Calendar is documented as refreshing only every 24 hours, which is too slow for a family dashboard.

### Cited Findings

- **Where to find the URL.** On a computer, open Settings → pick the calendar under "Settings for my calendars" → "Integrate calendar" → "Secret address in iCal format".
- **Privacy warning and reset.** Google says the address should be known only to you, and if it leaks you can generate a new one with **Reset**. Resetting breaks existing subscribers.
- **Work and school accounts.** An admin may have restricted access to this setting. This does not apply to consumer accounts.
- Sources for the three points above: [Google Calendar Help: Sync your calendar with computer programs](https://support.google.com/calendar/answer/37648?hl=en). The page as indexed looked several years old, so menu labels may have shifted.
- **No official refresh frequency.** I found no Google documentation of how often the exported ICS refreshes. The widely quoted "12–24 hours" figure is about **Google consuming** external ICS subscriptions, not Google's export feed. — [twocal](https://twocal.app/p/google-calendar-ics-refresh-delay/); [hetk.io](https://www.hetk.io/blog/how-to-subscribe-ics-calendar-google/)
- **When a feed stops updating.** If a feed suddenly stops, the secret address may have been reset; re-copy it. Treat the URL like a password. — [usecarly](https://www.usecarly.com/blog/how-to-get-google-calendar-ics-url/)
- **HA Remote Calendar behavior.**
  - It fetches the calendar at startup and retries on failure.
  - After that the update interval is **fixed at 24 hours**, with no UI option to change it.
  - It is read-only, and the entity state shows only the next event.
  - It supports HTTP Basic auth but no custom headers.
  — [Home Assistant: Remote Calendar](https://www.home-assistant.io/integrations/remote_calendar/) (search-extract; verify the interval)
- **HA Remote Calendar with Google URLs.** Some users reported "Invalid .ics file" errors with Google public and secret addresses. Another reported that a second remote calendar failed with "Service is already configured". — [HA Community: Invalid .ics with google link](https://community.home-assistant.io/t/remote-calendar-integration-setup-invalid-ics-with-google-link/873506); [HA Community: unable to add a second calendar](https://community.home-assistant.io/t/remote-calendar-unable-to-add-a-second-calendar/927076)
- **Community ICS alternative.** The HACS "ICS Calendar" custom component offers configurable options. — [HA Community: ICS Calendar platform](https://community.home-assistant.io/t/ics-calendar-platform/132023)
- **HA Google Calendar integration.** It uses OAuth (see section 1). The `google.create_event` action works only if the integration was granted read-write access in its options. — [HA: Google Calendar](https://www.home-assistant.io/integrations/google/); [HA action google.create_event](https://www.home-assistant.io/actions/google.create_event/)
- **Generic HA calendar action.** `calendar.create_event` works on entities that support the CREATE_EVENT feature. — [HA action calendar.create_event](https://www.home-assistant.io/actions/calendar.create_event/); [HA developer docs: Calendar entity](https://developers.home-assistant.io/docs/core/entity/calendar/)
- **ICS parsing libraries.**
  - **Node:** `node-ical` handles RRULE expansion, EXDATE and RECURRENCE-ID overrides, and has an `expandRecurringEvent()` helper. Version 0.27.3 was recent at search time. The older `ical` npm package (0.8.0) was last published about 7 years ago. — [node-ical GitHub](https://github.com/jens-maus/node-ical); [node-ical npm](https://www.npmjs.com/package/node-ical); [ical npm](https://www.npmjs.com/package/ical)
  - **Python:** `icalendar` parses but does not expand recurrences. `recurring-ical-events` adds expansion; release 3.8.2 came out on April 30, 2026. allenporter's `ical` does parsing plus timeline expansion and needs Python 3.11 or later. — [recurring-ical-events PyPI](https://pypi.org/project/recurring-ical-events/); [python-recurring-ical-events](https://github.com/niccokunzmann/python-recurring-ical-events); [allenporter/ical](https://github.com/allenporter/ical)

### Inferences

- The secret ICS URL suits a read-only, zero-maintenance first version, ideally fetched by the home server and not embedded in the tablet page. Polling every 5–15 minutes is fine. The data may still lag Google's UI by an unknown amount.
- Each calendar has its own secret URL, so a family view means collecting one URL per calendar. If any URL leaks (for example through a browser history sync or screenshot), reset that calendar's address.
- HA Remote Calendar's 24-hour refresh makes it a poor fit for same-day events. Prefer the HA Google Calendar integration (OAuth, polled) or your own ICS fetcher.
- Recurrence expansion (RRULE, EXDATE, moved instances) and time zones are where hand-rolled ICS parsing fails. Use node-ical, or Python `ical` / `icalendar` with `recurring-ical-events`. For Java, the leading language candidate in ADR-0005, ical4j is the usual choice (not researched here).

### Gaps

- No official Google statement on refresh latency of the exported secret ICS feed.
- Not confirmed whether a secret iCal address is offered for calendars *shared with* the owner (as opposed to owned) or what permission level that requires. Searches returned nothing authoritative.
- ical4j (Java) current status was not covered by search results.

## 3. Family members' calendars (shared calendars) and creating events from the tablet (write scope)

### Takeaway

The cleanest family setup is a shared "Family" secondary calendar plus each member's calendar shared with the owner's account. The dashboard then reads everything through the owner's single OAuth grant, via the calendar list. Creating events from the tablet needs a write scope (`calendar.events`, or the narrower `calendar.events.owned` / `calendar.app.created`) and the right sharing permission on the target calendar.

### Cited Findings

- **Sharing permission levels.** "Make changes and manage sharing" is Google's highest level. It includes everything in "Make changes to events" plus re-sharing and changing permissions. Recipients may need to enable a shared calendar in their list. — [Calendarbridge](https://calendarbridge.com/blog/how-to-create-a-shared-google-calendar/); [greminders](https://www.greminders.com/articles/changing-permissions-of-shared-google-calendars/) (secondary)
- **New 2026 access level.** `writerWithoutPrivateAccess` became generally available on June 29, 2026. It gives read and write access, but private events appear with details hidden. — [Calendar API release notes](https://developers.google.com/workspace/calendar/release-notes); [Acl reference](https://developers.google.com/workspace/calendar/api/v3/reference/acl)
- **Write scopes.**
  - `calendar.events.owned`: create, change and delete events on calendars you own.
  - `calendar.app.created`: limited to calendars the app itself creates.
  - `calendar.events`: view and edit events on all your calendars.
  — [OAuth 2.0 Scopes for Google APIs](https://developers.google.com/identity/protocols/oauth2/scopes); [Events: insert](https://developers.google.com/workspace/calendar/api/v3/reference/events/insert)
- **HA write access.** HA's `google.create_event` needs read-write access chosen in the integration options. Read-only blocks event creation. — [HA: google.create_event](https://www.home-assistant.io/actions/google.create_event/)
- **Community example.** A "DIY Family Calendar (Skylight)" HA dashboard shows a shared family calendar used with read-write access. — [HA Community: DIY Family Calendar (Skylight)](https://community.home-assistant.io/t/diy-family-calendar-skylight/844830)

### Inferences

- Read path: use `calendarList.list` (needs `calendar.readonly` or `calendar.calendarlist.readonly`), then `events.list` per calendar.
- Write path: if the tablet only adds to a dedicated "Family" calendar the owner created, `calendar.events.owned` is the narrowest fitting scope. Note that `calendar.app.created` only covers calendars the app itself created, so the app would have to create the Family calendar for that scope to work.
- ICS URLs are read-only. Any "add event from tablet" feature forces the OAuth path, or HA's Local Calendar.
- If a family member's calendar is shared "See only free/busy", the API returns no event details. The dashboard should handle that case.

### Gaps

- Not verified whether write access via the API is blocked when the owner has only "See all event details" on a shared calendar. It almost certainly is, but no source was retrieved.

## 4. Official Google Keep API: available to consumer accounts?

### Takeaway

No. As of 2026 the official Keep API is still documented as an enterprise-administration API for Google Workspace. Admins use it to create, list and delete notes and manage permissions, for example for CASB use cases. Personal @gmail.com accounts cannot use it.

### Cited Findings

- **What the API is for.** Google describes it as "a RESTful API used to create apps allowing enterprise administrators to manage Google Keep notes". Its use case is resolving issues flagged by cloud access security broker (CASB) software. — [Google Keep API overview](https://developers.google.com/keep/api/guides); [also at /workspace/keep](https://developers.google.com/workspace/keep/api/guides)
- **Workspace only.** Keep admin features (audit logs plus the API) "are only available for Google Workspace customers, and are not available for users with personal Google accounts." — [Google Workspace Updates, May 2021](https://workspaceupdates.googleblog.com/2021/05/keep-audit-logs-and-api.html)
- **How access is granted.** Access goes through admin-enabled domain-wide delegation with a service account. Make.com users report that Keep scopes don't appear without a Workspace Enterprise account. — [greatapis.com](https://greatapis.com/api/google-keep/); [Make Community](https://community.make.com/t/google-keep-api-access/84070)
- **2026 confirmation.** A 2026 write-up confirms that consumer integrations rely on the unofficial gkeepapi. — [usecarly (2026)](https://www.usecarly.com/blog/claude-google-keep-integration/)

### Inferences

- Any Keep-on-dashboard design for a consumer account depends on unofficial, reverse-engineered access (section 5) or on moving the household's lists elsewhere (section 6).

### Gaps

- None material; consistent across sources.

## 5. Unofficial Keep access: gkeepapi (Python), master-token / gpsoauth authentication, breakages 2024–2026, account risk, and HA integrations

### Takeaway

gkeepapi is still maintained (0.17.1, January 2026), but logging in is the fragile part. Google now blocks password and app-password logins. The working method is a browser "EmbeddedSetup" `oauth_token` cookie exchanged via gpsoauth for an Android "master token". That token has full access to the Google account. HA custom integrations built on gkeepapi exist and broke in 2025 on dependency pins. It is usable for a personal hobby project, but it is a security and stability liability.

### Cited Findings

- **Releases.** The latest PyPI release is **0.17.1 (January 5, 2026)**; 0.17.0 came out on November 7, 2025. There were GitHub commits in December 2025. — [gkeepapi on PyPI](https://pypi.org/project/gkeepapi/); [gkeepapi GitHub](https://github.com/kiwiz/gkeepapi); [releases](https://github.com/kiwiz/gkeepapi/releases)
- **Master tokens are dangerous.** gkeepapi's docs warn that master tokens have full access to your account and should be kept in a secrets store. — [gkeepapi docs/index.rst](https://github.com/kiwiz/gkeepapi/blob/main/docs/index.rst)
- **Password logins fail.** `perform_master_login()` with a password or app password "is unreliable and can return BadAuthentication". — [rossc719g/keep-mcp](https://github.com/rossc719g/keep-mcp)
- **App passwords stopped working too.** gkeepapi "cannot log in with just a password anymore – Google blocks that as suspicious". Users report app passwords stopped yielding master tokens ("Could not get master token"). — [chriiiish/home-assistant-google-keep](https://github.com/chriiiish/home-assistant-google-keep); [rithvikvibhu gist comments](https://gist.github.com/rithvikvibhu/952f83ea656c6782fbd0f1645059055d)
- **The cookie workaround.** gpsoauth documents an alternative flow for BadAuthentication: get the `oauth_token` cookie from accounts.google.com/EmbeddedSetup and call `exchange_token`. The cookie is single-use and short-lived. — [gpsoauth GitHub](https://github.com/simon-weber/gpsoauth); [gpsoauth PyPI](https://pypi.org/project/gpsoauth)
- **Token lifetime.** The master token reportedly lasts until the account password changes. It can be revoked from Google's device-activity page. — [feuerdev/keep-mcp listing](https://mcpservers.org/servers/feuerdev/keep-mcp) (secondary)
- **Caching caveat.** A February 2026 issue notes that the FAQ-suggested caching saves only the master token, not the auth token. — [gkeepapi issue #190](https://github.com/kiwiz/gkeepapi/issues/190)
- **Troubleshooting advice.** Re-approve via DisplayUnlockCaptcha after idle periods, and test from a different IP or user to isolate blocks. — [gkeepapi docs](https://github.com/kiwiz/gkeepapi/blob/main/docs/index.rst); [fcastilloec/gkeep-list-sync #36](https://github.com/fcastilloec/gkeep-list-sync/issues/36)
- **2025 breakage in HA.** The `google_keep_sync` custom integration broke in HA 2025.7 because pinned requirements (`gpsoauth==1.1.1`, `urllib3==1.26.18`, `gkeepapi==0.15.1`) failed to resolve. A beta version plus restarts fixed it. — [HA Community: Google Keep Sync error in 2025.7](https://community.home-assistant.io/t/google-keep-sync-error-in-2025-7/907319)
- **HA integrations built on gkeepapi.**
  - Bidirectional Keep list ↔ HA to-do sync (watkins-matt "google_keep_sync"; the repository link was truncated in results). — [HA Community feature thread](https://community.home-assistant.io/t/google-keep-integration-for-shopping-list-syncronization/634810?page=2)
  - chriiiish/home-assistant-google-keep: Keep lists → HA to-do, polling about every 60 seconds. — [GitHub](https://github.com/chriiiish/home-assistant-google-keep)
  - fcastilloec/gkeep-list-sync: Keep → HA Shopping List. — [GitHub](https://github.com/fcastilloec/gkeep-list-sync)
  - PiotrMachowski's Keep sensor (older). — [GitHub](https://github.com/PiotrMachowski/Home-Assistant-custom-components-Google-Keep)
- **No account-lock reports found.** I found no documented account suspensions or locks tied to gkeepapi. The only reported risk is login blocks and "suspicious sign-in" style rejections. — absence noted after searching [gkeepapi issues](https://github.com/kiwiz/gkeepapi)

### Inferences

- A master token is effectively a full-account credential, far broader than any OAuth scope. If the home server is compromised, the whole Google account (Gmail, Drive, Photos) is exposed, not just Keep. Mitigations: a dedicated household Google account that owns the shared Keep lists, shared to family members, rather than the owner's primary account; a secrets store; and revoking the device in Google Account when retired.
- Expect periodic breakage: login flow changes, Keep protocol changes, and Python dependency pins inside HA. Budget for manual re-authentication and do not make Keep the only path for critical lists.

### Gaps

- No official Google statement on gkeepapi or master-token use, and no reliable data on ban or lock risk.
- The current maintenance state and HA-version compatibility of watkins-matt's google_keep_sync in October 2026 were not confirmed.

## 6. Realistic alternatives for shared household notes and lists (Google Tasks, HA To-do, Todoist, Microsoft To Do, AnyList, Bring!, Alexa sync, Gemini/Keep)

### Takeaway

For an officially supported, consumer-account path:

- **Google Tasks API:** consumer-available, uses the same OAuth client as Calendar, and has a core HA integration.
- **HA Local To-do / Shopping list:** local, no cloud.
- **Bring!:** core HA integration, strong for shared grocery lists.
- **Todoist:** official API v1.

Alexa's list API for third parties ended on July 1, 2024; only fragile community workarounds remain. Gemini for Home still writes notes and lists to Google Keep in 2026, with a bug in August 2026 that Google says is fixed. Voice capture into Keep therefore works, but the dashboard cannot read Keep officially. This is the core tension.

### Cited Findings

- **Google Tasks API scopes.** `https://www.googleapis.com/auth/tasks` (create, edit, organize, delete) and `tasks.readonly` (view). Public apps using user-data scopes must be verified, and Google advises the narrowest scope. — [Choose Google Tasks API scopes](https://developers.google.com/workspace/tasks/auth); [also at /tasks/auth](https://developers.google.com/tasks/auth)
- **HA Google Tasks (core, since 2023.11).**
  - One HA to-do entity per Google task list; create, update and delete items.
  - Title maps to Title, description to Details.
  - Due dates are supported but **without time of day**, because the Tasks API lacks it.
  - Cloud polling.
  — [HA: Google Tasks](https://www.home-assistant.io/integrations/google_tasks/); [HA 2023.11 release](https://www.home-assistant.io/blog/2023/11/01/release-202311/)
- **Shared OAuth credentials in HA.** HA's docs say the same OAuth credentials are shared across Google integrations (Photos, Nest, Tasks, Mail) with redirect `https://my.home-assistant.io/redirect/oauth`. — [HA: Google Health](https://rc.home-assistant.io/integrations/google_health)
- **HA to-do platform.** It provides `todo.add_item` / `todo.get_items` and add, complete and remove triggers. The to-do list card can show lists from Bring!, Todoist, Local To-do and others. — [HA: To-do list](https://www.home-assistant.io/integrations/todo/); [HA: To-do list card](https://www.home-assistant.io/dashboards/todo-list/)
- **HA built-in Shopping list.** It is local and shown in the sidebar and to-do card. — [HA: Shopping list](https://www.home-assistant.io/integrations/shopping_list/)
- **Bring! (core HA integration).**
  - Exposes Bring! lists as to-do entities.
  - Adds sensors for urgent and discounted items and an activity event entity, and supports Assist voice.
  - Bring! is described as a shared grocery-list app for families.
  - Occasional 502 errors retry after 90 seconds.
  — [HA: Bring!](https://www.home-assistant.io/integrations/bring/)
  - Bring! reportedly has no official public API; the integration uses its app backend (search summary, unverified).
- **Todoist (core HA integration).** It exposes to-do and calendar entities. Completed items are not visible because the Todoist REST API doesn't return them. — [HA: Todoist](https://www.home-assistant.io/integrations/todoist/)
- **Todoist API changes.**
  - Todoist moved to a unified **API v1**. Old `/rest/v2/` endpoints redirect or return **HTTP 410**.
  - v1 brings string IDs and cursor pagination.
  - Personal tokens come from Todoist settings, and the API is free to use, with features gated by plan.
  — [Todoist API v1 docs](https://developer.todoist.com/api/v1/); [dashy issue #2025](https://github.com/lissy93/dashy/issues/2025); [airbyte PR #88183](https://github.com/airbytehq/airbyte/pull/88183)
  - Rate limit of 1,000 requests per user per 15 minutes — secondary only: [freeapihub](https://freeapihub.com/apis/todoist-api)
- **Microsoft To Do (Graph).** Delegated `Tasks.Read` / `Tasks.ReadWrite` are available for **personal Microsoft accounts** without admin consent (for example `POST /me/todo/lists`, `/me/todo/lists/{id}/tasks`). The Graph Toolkit To Do component is deprecated: retirement began September 1, 2025, with full retirement on August 28, 2026. — [Create todoTaskList](https://learn.microsoft.com/en-us/graph/api/todo-post-lists?view=graph-rest-1.0); [Create todoTask](https://learn.microsoft.com/en-us/graph/api/todotasklist-post-tasks?view=graph-rest-1.0); [Graph permissions reference](https://learn.microsoft.com/en-us/graph/permissions-reference); [MGT To Do component](https://learn.microsoft.com/en-us/graph/toolkit/components/todo)
- **AnyList.** No official API. The community HACS integration `moryoav/ha-anylist` (v0.4.3) exposes AnyList lists as to-do entities and supports recipe actions. — [HA Community: AnyList for Home Assistant](https://community.home-assistant.io/t/anylist-for-home-assistant-shopping-lists-recipes-and-categories/1014626)
- **AnyList and Alexa.** After the Alexa change, AnyList uses its own skill ("tell AnyList…"). — [criticalx substack](https://criticalx.substack.com/p/is-anylist-via-alexa-dead); [listee.in](https://www.listee.in/blog/family-shopping-list-app-alexa) (secondary)
- **Alexa list API ended.** Amazon ended third-party access to Alexa shopping and to-do lists on **July 1, 2024**, retiring List Skills and the List Management REST API. Unnamed commands now only affect Alexa's own list. — [listee.in](https://www.listee.in/blog/family-shopping-list-app-alexa); [mealdill](https://www.mealdill.com/blog/grocery-list-app-alexa); [HA Community: Sync Alexa Shopping List](https://community.home-assistant.io/t/sync-alexa-shopping-list-with-home-assistant/775763). These are secondary; Amazon's notice and the original Verge report were not retrieved.
- **Community Alexa-sync workarounds.**
  - `ocean90/alexa_shopping_sync` (HACS): two-way sync between Alexa's shopping list and any HA to-do entity. It needs an Amazon login plus the 2FA authenticator secret. — [GitHub](https://github.com/ocean90/alexa_shopping_sync)
  - A newer integration reverse-engineered from the Alexa app's traffic. — [HA Community](https://community.home-assistant.io/t/alexa-shopping-to-do-lists-integration/1007262)
  - Browser or Selenium scrapers. — [madmachinations](https://github.com/madmachinations/home-assistant-alexa-shopping-list); [thiagobucca CDP](https://github.com/thiagobucca/ha-alexa-shopping-list-sync)
  - A one-way Alexa → Bring! blueprint using Alexa Media Player. — [HA Community](https://community.home-assistant.io/t/sync-alexa-shopping-list-one-way-to-bring/825623)
  - `lonlazer/ha-alexa-todo-lists` is marked retired. — [GitHub](https://github.com/lonlazer/ha-alexa-todo-lists)
- **Gemini for Home and Keep in 2026.**
  - Google's support page says basic notes, lists and reminders in Gemini for Home are available at no cost. — [Google Home Help: Gemini for Home](https://support.google.com/googlehome/answer/16618650?hl=en)
  - In August 2026 a Google Home bug broke "add X to my Y list" to Keep for some users; some found the notes-and-lists toggle disabled. — [9to5Google, Aug 25 2026](https://9to5google.com/2026/08/25/google-home-keep-integration-broken-due-to-bug/); [Yahoo Tech](https://tech.yahoo.com/ai/gemini/articles/google-home-bug-disrupts-keep-195000605.html)
  - Android Authority reports Google said the Keep list bug was resolved, in the September 2026 Gemini for Home upgrade coverage. — [Android Authority, Sept 2026](https://www.androidauthority.com/google-home-gemini-upgrades-september-2026-3706505/)
  - A spring 2026 update improved natural-language list edits. — [Droid Life, Apr 2026](https://www.droid-life.com/2026/04/13/gemini-for-google-home-keeps-getting-huge-upgrades/)

### Inferences

- **Best fit if voice capture via Google or Nest speakers matters:** keep using Keep for voice capture and read it with gkeepapi (accepting the risks in section 5). Or move the household's lists to a system HA can read officially. Google Tasks is the officially accessible Google option, but I found no 2026 source confirming that Gemini for Home writes shopping lists to Google Tasks. Keep is its documented target.
- **Best fit for a shared grocery list with phone apps for every family member:** Bring! (core HA integration, family sharing) or AnyList (custom HACS, US-popular, recipe features). HA's Local To-do or Shopping list is the most robust but has no polished mobile app beyond the HA companion app.
- **Alexa households:** no supported sync exists since July 2024. Community HACS integrations need Amazon credentials or 2FA secrets and break with Amazon changes.

### Gaps

- No HA core integration for Microsoft To Do was found (only Graph API docs). Custom components may exist; not verified.
- Whether Gemini for Home can be pointed at Google Tasks or a third-party list provider in 2026 is unconfirmed.
- The sensitivity tier of the Tasks scopes was not confirmed (likely sensitive, like Calendar).
- Bring!'s API terms and the stability of the HA integration's backend access were not verified.

## 7. Gmail on the dashboard (brief): restricted scopes, verification and CASA, and lighter alternatives (Atom feed, IMAP with app passwords)

### Takeaway

An **unread count** is achievable with the non-sensitive `gmail.labels` scope: `users.labels.get` on INBOX returns `messagesUnread` / `threadsUnread`. **Subjects or senders of important emails** need `gmail.metadata` or `gmail.readonly`, which are restricted scopes. For a public app those require verification plus an annual third-party security assessment (CASA). For a personal, unverified app under 100 users, the same personal-use exemption appears to apply, with warnings. IMAP with a 16-character app password (2-Step Verification required) still exists but Google calls it "not recommended". The Gmail Atom feed is still documented, now described as Workspace-oriented and OAuth-preferred.

### Cited Findings

- **Restricted Gmail scopes.** `https://mail.google.com/`, `gmail.readonly`, `gmail.compose`, `gmail.insert`, `gmail.modify`, `gmail.metadata`, `gmail.settings.basic`, `gmail.settings.sharing`. `gmail.send` is sensitive. — [Choose Gmail API scopes](https://developers.google.com/workspace/gmail/api/auth/scopes); [Workspace user data and developer policy](https://developers.google.com/workspace/workspace-api-user-data-developer-policy)
- **Restricted-scope requirements.**
  - Verification plus a security assessment, with a 6-week expected timeline.
  - Apps that store or transmit restricted data on servers need the assessment, generally yearly.
  - Unverified apps using restricted scopes are subject to a 100-new-user cap.
  — [Restricted scope verification](https://developers.google.com/identity/protocols/oauth2/production-readiness/restricted-scope-verification)
- **Local apps not obviously exempt.** Secondary sources say "local-only" does not automatically waive CASA for production apps. — [Google Security community](https://security.googlecloudcommunity.com/google-security-operations-2/what-happened-to-local-app-gmail-api-access-7138); [Nylas](https://developer.nylas.com/docs/provider-guides/google/google-verification-security-assessment-guide/)
- **Personal-use exemption.** Google's "When is verification not needed" page lists personal use under 100 users as exempt; the extract did not separate sensitive from restricted scopes. — [Google Cloud Help](https://support.google.com/cloud/answer/13464323?hl=en)
- **Unread count without restricted scopes.** `users.labels.get` accepts the `gmail.labels` scope. The Label resource has `messagesUnread` and `threadsUnread`. Google's labels guide treats `gmail.labels` as the label-management scope, and third parties classify it as non-sensitive. — [users.labels.get](https://developers.google.com/workspace/gmail/api/reference/rest/v1/users.labels/get); [users.labels resource](https://developers.google.com/workspace/gmail/api/reference/rest/v1/users.labels); [Manage labels](https://developers.google.com/workspace/gmail/api/guides/labels); [Unipile scopes guide](https://www.unipile.com/gmail-api-scopes-guide/)
- **Gmail inbox (Atom) feed.**
  - Still documented, last updated March 25, 2025 (German page).
  - OAuth 2.0 is the preferred authentication.
  - The English page states "This feed is only available for Gmail accounts on Google Workspace domains."
  — [Gmail Inbox Feed](https://developers.google.com/workspace/gmail/gmail_inbox_feed); [de version](https://developers.google.com/gmail/gmail_inbox_feed?hl=de)
  - A 2026 developer post reports it still works for personal accounts in a signed-in browser and returns unread mail only. — [dev.to](https://dev.to/just_commit/gmail-has-a-hidden-atom-feed-i-built-a-chrome-extension-on-it-with-no-oauth-and-no-backend-tags-523o) (anecdotal)
- **Password-only access retired.** Google wound down password-only "less secure apps" access in 2024–2025. The dates were revised several times: September 30, 2024, then March 14, 2025 and May 1, 2025 for Workspace. For personal accounts, IMAP works over OAuth. — [Workspace Updates](https://workspaceupdates.googleblog.com/2023/09/winding-down-google-sync-and-less-secure-apps-support.html); [Workspace Admin Help: Transition from LSA](https://support.google.com/a/answer/14114704?hl=en)
- **App passwords still exist.** They are 16-digit passcodes that require 2-Step Verification, described as "not recommended". — [Gmail Help: Sign in with app passwords](https://support.google.com/mail/answer/185833?hl=en)

### Inferences

- **Recommendation:** for an unread badge, use the Gmail API with `gmail.labels` only, on the same unverified-production OAuth client as Calendar, and poll INBOX (or a "dashboard" label set by a Gmail filter) for `messagesUnread`. This avoids restricted scopes entirely.
- **If subjects are needed:**
  - Option A: `gmail.metadata` on the personal unverified client (warning screen, personal-use exemption; the CASA trigger appears tied to verification or production beyond 100 users, but this is not definitively confirmed).
  - Option B: IMAP plus an app password on the home server. It is simple, but the app password grants full mailbox access over IMAP and Google discourages it.
- Avoid the Atom feed for server-side use. Basic-auth access to it ended with less-secure-apps, and Google now documents it as Workspace-oriented.

### Gaps

- No Google page was retrieved that explicitly confirms the personal-use exemption covers **restricted** Gmail scopes in production, unverified.
- The sensitivity tier of the Atom feed scope `https://mail.google.com/mail/feed/atom` was not confirmed.

## 8. National Weather Service API (api.weather.gov): access, endpoints, alerts, reliability, and NOAA radar imagery

### Takeaway

api.weather.gov is free, needs no key (only an identifying User-Agent, ideally with contact info), and gives US-quality data:

- 12-hour-period and hourly forecasts per 2.5 km gridpoint;
- raw gridded data;
- observations;
- active alerts (CAP/GeoJSON).

It has a long record of intermittent 500 and 503 errors on gridpoint forecasts, occasional planned gridpoint outages, and observation lags. Design for retries, cache to the `Expires` header, and keep a fallback provider. NOAA also publishes free radar WMS and ArcGIS services.

### Cited Findings

- **User-Agent rule.** "A User Agent is required to identify your application… If you include contact information (website or email), we can contact you if your string is associated to a security event. This will be replaced with an API key in the future." — [NWS API documentation](https://www.weather.gov/documentation/services-web-api)
- **Missing User-Agent.** Third parties report 401 or 403 responses without a User-Agent (conflicting codes). — [MIT App Inventor community](https://community.appinventor.mit.edu/t/unable-to-access-national-weather-service-api/66547); [nwsapy docs](https://nwsapy.readthedocs.io/en/stable/gettingstarted.html)
- **Endpoint flow.** Call `/points/{lat},{lon}` first; it links to forecast, forecastHourly, observation stations and the gridpoint. `/gridpoints/{wfo}/{x},{y}` gives raw gridded data, with `/forecast` and `/forecast/hourly` under it. — [NWS API documentation](https://www.weather.gov/documentation/services-web-api); [forecast-v3 docs](https://forecast-v3.weather.gov/documentation)
- **Alerts.** `/alerts/active` can be filtered by point, zone or area, plus severity and urgency. Guidance is no more than one request every 30 seconds. "Rate limiting firewalls are in place". — [NWS Alerts Web Service](https://www.weather.gov/documentation/services-web-alerts)
- **Rate limits are unpublished.** A maintainer says they "are generally liberal enough to handle the majority of use cases".
- **Caching.** Re-requesting before `Expires` returns the same data. `/alerts/active` has a max-age of 30 seconds; gridpoint forecast about 1 hour.
- Source for the two points above: [weather-gov/api discussion #719](https://github.com/weather-gov/api/discussions/719)
- **No multi-gridpoint requests.** — [discussion #429](https://github.com/weather-gov/api/discussions/429)
- **Reliability history.**
  - Recurring "Status 500: Unexpected Problem" on gridpoint forecasts. — [discussion #241](https://github.com/weather-gov/api/discussions/241)
  - NWS warned the gridpoints endpoint might be disabled from December 22, 2024 to January 1, 2025 (503s). — [PNS 24-66](https://www.weather.gov/media/notification/pdf_2023_24/pns24-66_potential_api_outage_aaa.pdf)
  - A 2025 report of observations lagging more than an hour and history cut to about 4 days. — [discussion #751](https://github.com/weather-gov/api/discussions/751)
  - Missing dewpoint and RH in the forecast. — [discussion #752](https://github.com/weather-gov/api/discussions/752)
  - An October 2026 single report that the alerts Atom URL returns HTTP 400. — [jelrod27 issue #659](https://github.com/jelrod27/Weather-application-/issues/659) (unconfirmed)
- **HA NWS integration.** The "API key" field accepts any string; an email is suggested. The METAR station is optional and defaults to the nearest; three-letter codes need a K prefix. Each entry needs a unique lat/lon. — [HA: National Weather Service](https://www.home-assistant.io/integrations/nws/); [HA core issue #159221](https://github.com/home-assistant/core/issues/159221)
- **NOAA radar services.**
  - `radar_base_reflectivity_time` (NWS ArcGIS ImageServer) supports time-enabled WMS 1.3.0 over a 4-hour window, refreshes about every 10 minutes, uses UTC times in epoch ms, and defaults to the latest image. — [mapservices.weather.noaa.gov ImageServer](https://mapservices.weather.noaa.gov/eventdriven/rest/services/radar/radar_base_reflectivity_time/ImageServer); [WMS capabilities](https://mapservices.weather.noaa.gov/eventdriven/services/radar/radar_base_reflectivity_time/ImageServer/WMSServer?request=GetCapabilities&service=WMS)
  - A non-time-enabled `radar_base_reflectivity` MapServer refreshes every 5 minutes. — [MapServer](https://mapservices.weather.noaa.gov/eventdriven/rest/services/radar/radar_base_reflectivity/MapServer)
  - nowCOAST GeoServer WMS `conus_base_reflectivity_mosaic` (MRMS, 1 km, about every 4 minutes). — [nowCOAST WMS capabilities](https://nowcoast.noaa.gov/geoserver/observations/weather_radar/wms?SERVICE=WMS&REQUEST=GetCapabilities)

### Inferences

- NWS should be the primary US forecast and **alerts** source. Alerts are its unique value; no free commercial API matches official NWS warnings.
- Cache `/points` results indefinitely (gridpoint mapping rarely changes). Fetch the forecast hourly and alerts every 1–5 minutes, respecting `Expires`. On a 5xx, keep showing cached data with a "stale" indicator and fall back to Open-Meteo.

### Gaps

- No official NWS uptime or SLA figures.
- Whether the "API key in the future" plan has a date (none found).
- The current status of `opengeo.ncep.noaa.gov` GeoServer endpoints was not confirmed.

## 9. Other forecast APIs: Open-Meteo, Pirate Weather, Apple WeatherKit REST, OpenWeatherMap One Call 3.0, Tomorrow.io, AccuWeather (2026 terms, limits, data notes)

### Takeaway

- **Open-Meteo:** the best no-key free option for a non-commercial personal dashboard. Limits are under 10k calls/day, 5k/hour and 600/minute; CC BY 4.0 attribution; HRRR 15-minute data for the US.
- **Pirate Weather:** free key, 10k calls/month (about 1 call every 4.5 minutes), Dark Sky-compatible.
- **WeatherKit REST:** 500k calls/month included with the $99/year Apple Developer Program; JWT auth and attribution required.
- **OpenWeatherMap One Call 3.0:** 1,000 calls/day free but requires a billing subscription; cap it to avoid charges.
- **Tomorrow.io free:** 500/day, 25/hour.
- **AccuWeather:** ended its free tier in 2025 (14-day trial, then paid Starter).

### Cited Findings

- **Open-Meteo.**
  - Free API for non-commercial use: "Less than 10'000 API calls per day, 5'000 per hour and 600 per minute". No key.
  - CC BY 4.0 attribution required.
  - It may block misusing apps and IPs without notice.
  - Commercial use (subscriptions, ads, commercial products) needs a paid API key.
  — [Open-Meteo Terms](https://open-meteo.com/en/terms); [Pricing](https://open-meteo.com/en/pricing); [Home](https://open-meteo.com/)
- **Open-Meteo data sources.**
  - "Best match" combines global ICON, GFS and IFS with regional models. For the US and Canada, NOAA HRRR is layered on, with 15-minutely data (3 km, 18-hour horizon).
  - NBM for CONUS is listed at 2.5 km on one page and 3 km on another (conflict).
  — [Open-Meteo GFS & HRRR API](https://open-meteo.com/en/docs/gfs-api); [Docs](https://open-meteo.com/en/docs); [Historical Forecast API](https://open-meteo.com/en/docs/historical-forecast-api)
- **HA Open-Meteo integration.** No account or key; "free for open-source and non-commercial use"; forecasts per HA zone. — [HA: Open-Meteo](https://www.home-assistant.io/integrations/open_meteo/)
- **Pirate Weather.**
  - Free tier is 10,000 calls/month; a $2/month donation raises it to 20,000. Keys come via Apiable.
  - Older 2021 posts said 20,000 free (conflict; outdated).
  — [pirateweather.net](https://pirateweather.net/); [Pirate Weather v0.1 blog](https://docs.pirateweather.net/en/latest/Blog/indexv01/); [GitHub issue #320](https://github.com/Pirate-Weather/pirateweather/issues/320)
  - The HA integration is installed via HACS custom repository and set up in the UI; docs advise update intervals of at least 900 seconds to avoid running out of quota. — [Pirate Weather HA docs](https://docs.pirateweather.net/en/latest/ha/)
- **Apple WeatherKit.**
  - "Up to 500,000 API calls a month per Apple Developer Program membership"; REST is usable for websites and other platforms. Extra capacity is bought by the Account Holder.
  - 2022 launch tiers: $49.99/month for 1M calls up to $999.99/month for 20M (may be outdated).
  — [Get started with WeatherKit](https://developer.apple.com/weatherkit/get-started/); [9to5Mac 2022](https://9to5mac.com/2022/09/12/developers-weatherkit-api-requests/)
  - Program fee is $99 USD per membership year. — [Apple Developer Program](https://developer.apple.com/programs/); [Membership details](https://developer.apple.com/programs/whats-included/)
  - REST auth needs a WeatherKit-enabled private key plus a Services ID to sign JWT developer tokens. — [Apple: Create a services identifier and private key for WeatherKit](https://developer.apple.com/help/account/capabilities/create-a-services-identifier-and-private-key-for-weatherkit/); [forum thread](https://developer.apple.com/forums/thread/707418)
  - Attribution (Apple Weather mark plus legal link) is required for REST as well as Swift. — [WWDC22 Meet WeatherKit](https://developer.apple.com/videos/play/wwdc2022/10003/); [attribution forum](https://developer.apple.com/forums/thread/708013)
- **OpenWeatherMap One Call 3.0.**
  - "Pay as you call", with 1,000 calls/day free, then a per-call charge. You must subscribe via a billing form. The default daily cap is 2,000; users set it to 1,000 to stay free.
  — [One Call API 3.0](https://openweathermap.org/api/one-call-3); [Pricing](https://openweathermap.org/price); [Dark Sky migration guide](https://openweathermap.org/api/darksky-openweather-3); [HA Community: OWM 3.0](https://community.home-assistant.io/t/openweathermap-version-3-0/717870)
  - Overage is 0.0012 GBP/call, according to the One Call 2.5 deprecation page and IONOS. — [one-call-1-deprecated](https://openweathermap.org/one-call-1-deprecated); [IONOS](https://www.ionos.com/digitalguide/websites/web-development/openweather-api-key.md)
  - Whether a card is strictly mandatory was not confirmed.
- **Tomorrow.io.**
  - Free plan: 500 requests/day (reset 00:00 UTC), 25/hour, 3/second; map tiles count as requests; limited locations.
  - The official article is about 16 months old; 2026 comparisons repeat the same numbers.
  — [Tomorrow.io: Free API Plan Rate Limits](https://support.tomorrow.io/hc/en-us/articles/20273728362644-Free-API-Plan-Rate-Limits); [Tomorrow.io pricing overview](https://support.tomorrow.io/hc/en-us/articles/23554984091156-Tomorrow-io-Pricing-Overview)
- **AccuWeather.**
  - It retired its free limited trials with a new developer portal (July 2025 announcement). Replacements are a 14-day Core Weather trial (500 calls/day) or MinuteCast trial, then a paid Starter plan.
  - On September 9, 2025 the new API launched; old keys expired, and the new keys are incompatible with HA's integration. HA states AccuWeather "currently does not offer a free plan".
  - A 2026 aggregator claims a 50-calls/day free Developer plan (conflict, unverified).
  — [alternativeto news](https://alternativeto.net/news/2025/7/accuweather-to-end-free-core-weather-api-access-with-new-portal-launch/); [HA alert: AccuWeather API keys expired](https://alerts.home-assistant.io/alerts/accuweather/); [HA core issue #149554](https://github.com/home-assistant/core/issues/149554); [Michael Tsai](https://mjtsai.com/blog/2025/08/04/accuweather-to-discontinue-free-api/); [apispine (aggregator)](https://apispine.com/accuweather/pricing)

### Inferences

- **Budget math for one dashboard:**
  - Polling every 10 minutes is 4,320 calls/month, or 144/day. That fits Open-Meteo, OWM free (1,000/day), Tomorrow.io (500/day, 25/hour; 10-minute polling is 6 per hour), Pirate Weather (10k/month) and WeatherKit easily.
  - Pirate Weather at 5-minute polling is 8,640/month, close to its cap. Hourly plus daily in one call keeps it to 1 call per poll.
- **Best "no account" combination:** NWS (official forecast text plus alerts) and Open-Meteo (hourly and 15-minute numeric series, precipitation probability, global fallback). WeatherKit is attractive only if the owner already pays for Apple Developer. Its JWT signing must live on the server, since the private key must not ship to the tablet.
- **Avoid:** AccuWeather (no free tier) and OWM unless a billing cap is set. OWM's free tier needs a payment-backed subscription.
- **"Non-commercial" fit:** a private household dashboard plausibly fits Open-Meteo's terms. A product would not.

### Gaps

- Current 2026 WeatherKit overage prices, Pirate Weather's free tier, and Tomorrow.io limits were confirmed only via older official pages or aggregators.
- No independent 2025–2026 accuracy comparison of these providers for US locations was found.

## 10. Radar map tiles for a dashboard: RainViewer status, NOAA/NWS WMS, Iowa Environmental Mesonet

### Takeaway

RainViewer's free API was cut back on January 1, 2026:

- no nowcast (future) frames;
- no satellite;
- only the "Universal Blue" color scheme;
- maximum zoom 7, with HTTP 200 placeholder tiles above that;
- 100 requests per IP per minute.

For a US home view (zoom 8–10), Iowa Environmental Mesonet (IEM) NEXRAD XYZ or WMS tiles and NOAA's time-enabled WMS are better free choices. All are keyless.

### Cited Findings

- **RainViewer January 1, 2026 changes.** Nowcast discontinued, satellite IR discontinued, all color schemes except Universal Blue discontinued, maximum zoom set to 7, rate limit 100 requests/IP/minute. — [RainViewer API Transition Summary](https://www.rainviewer.com/api/transition-faq.html); [Weather Maps API](https://www.rainviewer.com/api/weather-maps-api.html)
- **RainViewer zoom-7 workaround.** Developers report a 200-status "Zoom Level Not Supported" PNG above z7; they cap layers with `maxNativeZoom: 7` and upscale. — [gsoc-monitor PR #32](https://github.com/alecbibat/gsoc-monitor/pull/32); [pi-weather-station PR #396](https://github.com/thicla01/pi-weather-station/pull/396); [Flowx forum](https://forum.flowx.io/t/rainviewer-will-discontinue-jan-2026/1993?page=4)
- **IEM tiles.**
  - The OGC page lists `nexrad-n0q` (current base reflectivity) and `nexrad-n0q-mXXm` (5–55 minutes old) for animation, plus `ridge::` single-site and mosaic layers and timestamped archives. — [IEM OGC Web Services](https://mesonet.agron.iastate.edu/ogc/); [IEM GIS RADAR](https://mesonet.agron.iastate.edu/GIS/ridge.phtml)
  - Tile template `https://mesonet.agron.iastate.edu/cache/tile.py/1.0.0/nexrad-n0q-900913/{z}/{x}/{y}.png`: free, keyless, about 5-minute refresh. — [WeatherTeam6 issue #214](https://github.com/TimHogan-HD/WeatherTeam6/issues/214) (third-party claim)
  - A Leaflet WMS example uses IEM. — [RStudio leaflet basemaps](https://rstudio.github.io/leaflet/articles/basemaps.html)
- **NOAA WMS services.** See section 8: NWS ArcGIS time-enabled WMS (4-hour window, about 10-minute refresh) and the nowCOAST MRMS mosaic (1 km, about 4 minutes).

### Inferences

- **Recommended:** a Leaflet or MapLibre map on the tablet with IEM `nexrad-n0q` XYZ tiles (or NOAA's time-enabled WMS for loops), a light basemap, and an attribution line. Use RainViewer only for a zoomed-out regional view, and not at all if future-radar was the goal.
- Frame-based animation: use the `-m05m … -m55m` IEM layers to build a 1-hour loop with about 12 tile requests per tile position. Preload frames to avoid flicker on a low-power tablet.

### Gaps

- IEM's formal usage policy or rate limits for embedded tile use were not retrieved.
- Whether RainViewer changed its free API again after January 2026 is unknown.

## 11. Home Assistant weather integrations (Met.no default, NWS, Open-Meteo, Pirate Weather) and the forecast service change (weather.get_forecasts)

### Takeaway

HA ships Met.no as the default weather integration, tracking the home location. NWS and Open-Meteo are core integrations needing no real key; Pirate Weather is a HACS custom integration needing a free key. Since 2024.3, weather entities no longer carry a `forecast` attribute. Dashboards must call the `weather.get_forecasts` action (daily, hourly or twice_daily) and read the response, for example via a trigger-based template sensor or the websocket `weather/subscribe_forecast`.

### Cited Findings

- **Met.no.** By default it uses the HA home location. Add more instances for other coordinates; each gets its own entity. Instances tracking home update when the home location changes. The default entity is often `weather.home`; an hourly entity can be enabled. — [HA: Met.no](https://www.home-assistant.io/integrations/met/); [HA Community](https://community.home-assistant.io/t/met-no-hourly-forecast-entity-not-available-in-latest-ha/616504)
- **NWS and Open-Meteo.** See sections 8 and 9. — [HA: NWS](https://www.home-assistant.io/integrations/nws/); [HA: Open-Meteo](https://www.home-assistant.io/integrations/open_meteo/)
- **Pirate Weather.** HACS install, UI setup with key, lat/lon, interval of at least 900 seconds, and a choice of weather and/or sensor entities. — [Pirate Weather HA docs](https://docs.pirateweather.net/en/latest/ha/)
- **Forecast attribute removal.** The 2023.9 release deprecated the forecast state attribute, "will be removed in Home Assistant Core 2024.3", replaced by a service (initially `weather.get_forecast`, now `weather.get_forecasts`). Users confirm removal in 2024.3; one says 2024.4 (conflict, minor). — [HA 2023.9 release notes](https://www.home-assistant.io/blog/2023/09/06/release-20239/); [HA Community: Removed deprecated forecast attribute](https://community.home-assistant.io/t/removed-deprecated-forecast-attribute-from-weatherentity/727347); [HA Community: where did the forecasts array go](https://community.home-assistant.io/t/where-did-the-forecasts-array-go-in-the-built-in-weather/723538)
- **Using the response.** It is returned in a `response_variable`. The common pattern is a trigger-based template sensor that copies the forecast into an attribute. — [HA Community: how to use weather.get_forecasts in a helper](https://community.home-assistant.io/t/how-to-use-weather-get-forcasts-in-a-helper/652380); [Blueprints thread](https://community.home-assistant.io/t/blueprints-templates-and-how-to-access-weather-forecasts-via-service-call/727066)
- **AccuWeather in HA.** HA's AccuWeather integration needs paid keys since September 2025 (section 9). — [HA alert](https://alerts.home-assistant.io/alerts/accuweather/)

### Inferences

- If the dashboard is an HA dashboard, use HA's own weather-forecast card, which subscribes to forecasts internally. If it is a custom web page reading HA's API, it must call `weather.get_forecasts` (REST `POST /api/services/weather/get_forecasts?return_response`) or subscribe via websocket. Reading the `forecast` attribute will return nothing.
- For US users, the NWS integration gives NWS forecasts in HA. Alerts are not part of the core NWS weather entity (not verified). The community "NWS Alerts" HACS integration is commonly used but was not researched here.

### Gaps

- Whether the core NWS integration exposes alerts was not verified.
- The HA `weather/subscribe_forecast` websocket API details were not retrieved.

## 12. Recommendation per data source (synthesis of sections 1–11)

### Takeaway

- **Calendar:** Calendar API via one personal OAuth client (published, unverified), polled by the home server or the HA Google Calendar integration. Use the secret ICS URL only for a read-only MVP.
- **Lists:** move shared lists to an officially integrable service (Google Tasks, or Bring! / AnyList for groceries, or HA Local To-do). Use gkeepapi only if keeping Keep, and Gemini voice capture into Keep, is non-negotiable, with a dedicated account and master-token hygiene.
- **Gmail:** `gmail.labels` unread count only.
- **Weather:** NWS (forecast plus alerts) plus Open-Meteo (numeric series and fallback) plus IEM radar tiles.

### Cited Findings

- **Calendar:** publishing the app removes the 7-day expiry; personal use is exempt from verification; the 100-user lifetime cap applies. — [Google Cloud Help: When is verification not needed](https://support.google.com/cloud/answer/13464323?hl=en); [Unverified apps](https://support.google.com/cloud/answer/7454865?hl=en); [HA Google Calendar](https://www.home-assistant.io/integrations/google/)
- **ICS:** bearer-URL privacy model with reset. — [Google Calendar Help](https://support.google.com/calendar/answer/37648?hl=en)
- **Keep:** the official API is Workspace-admin only; gkeepapi needs a full-access master token. — [Keep API overview](https://developers.google.com/keep/api/guides); [gkeepapi docs](https://github.com/kiwiz/gkeepapi/blob/main/docs/index.rst)
- **Tasks:** consumer OAuth scopes and core HA integration. — [Tasks scopes](https://developers.google.com/workspace/tasks/auth); [HA Google Tasks](https://www.home-assistant.io/integrations/google_tasks/)
- **Bring!:** core HA integration. — [HA Bring!](https://www.home-assistant.io/integrations/bring/)
- **Alexa:** the list API ended July 1, 2024. — [listee.in](https://www.listee.in/blog/family-shopping-list-app-alexa)
- **Gmail:** `gmail.labels` covers `labels.get` (with `messagesUnread`). — [users.labels.get](https://developers.google.com/workspace/gmail/api/reference/rest/v1/users.labels/get)
- **Weather:** NWS is keyless with a User-Agent; Open-Meteo is free non-commercial at under 10k calls/day; RainViewer was cut back to z7 in 2026; IEM tiles are keyless. — [NWS API](https://www.weather.gov/documentation/services-web-api); [Open-Meteo terms](https://open-meteo.com/en/terms); [RainViewer transition](https://www.rainviewer.com/api/transition-faq.html); [IEM OGC](https://mesonet.agron.iastate.edu/ogc/)

### Inferences

- **Architecture implication:** all Google credentials (OAuth refresh tokens, any gkeepapi master token, ICS secret URLs) and any WeatherKit private key should sit on the home server or in HA. The tablet page should only call a local backend or HA. A wall tablet is physically exposed and its browser state is easy to inspect.
- **Decision points for ADRs:**
  1. Calendar via OAuth versus ICS (write needs OAuth).
  2. Keep via gkeepapi versus migrating lists to Tasks, Bring! or AnyList.
  3. Weather provider pair.
  4. Whether the dashboard is HA-native (HA cards and integrations) or a custom page reading HA's API. If custom, it must handle `weather.get_forecasts`.
- **Assumptions to state:** single household, under 10 Google accounts consenting; non-commercial use (needed for Open-Meteo's free tier); US location (NWS coverage).

### Gaps

- No hands-on validation was possible in this session. Rules that change frequently should be re-checked against the Google Cloud Console at build time: Google scope tiers, Calendar API 2026 billing, RainViewer and AccuWeather terms.
