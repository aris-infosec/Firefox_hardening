# Firefox Hardening Guide for Security Engineers

A balanced, practical hardening profile for Mozilla Firefox.

Goals:
- Reduce passive tracking
- Minimize fingerprinting
- Lower attack surface
- Improve isolation
- Stay usable — no broken sites, no unsafe workarounds

---

## Philosophy: Why "Less Is More"

When I started hardening my browser, my instinct was to disable everything and install every privacy extension available. Over time I learned this is a common mistake — and it backfires.

Why it backfires:
- A default Firefox install looks like millions of other identical installs.
- Every unusual change (10 extensions, spoofed user-agent, JavaScript disabled) makes you stand out more, not less.
- Trackers exploit uniqueness, not just data collection.
- A heavily modified browser can be **more** identifiable than a stock one.

**Core principle: conformity is camouflage.**
- The goal isn't removing every signal.
- The goal is matching a large crowd of other users.
- This guide favors high-value changes that don't make you stand out.
- Habits (compartmentalization, separate profiles) matter more than any single setting.

---

## Part 1 — Choosing Your Browser

Before touching settings: why Firefox, and not something else?

| Browser | Engine | Strengths | Weaknesses |
|---|---|---|---|
| **Firefox** | Gecko (non-Chromium) | Last major independent engine; full extension support; deep `about:config` control; public security disclosure process | Telemetry on by default; 2025 Terms of Use drew criticism; needs manual hardening |
| **LibreWolf** | Gecko (Firefox fork) | Hardened by default — telemetry stripped, `resistFingerprinting` pre-enabled, uBlock Origin often bundled | Smaller team than Mozilla; manual updates on some platforms; stricter defaults can break sites |
| **Brave** | Chromium | Strong blocking out of the box (97%+ of trackers in independent tests); built-in Tor tab mode; fast | Shares Google's rendering engine; subject to Chromium's Manifest V3 limits; built-in crypto/wallet features add unused attack surface |
| **Vivaldi** | Chromium | Deep customization; built-in tracker/ad blocker, mail, calendar, notes; no user tracking | Chromium engine; fingerprint resistance weaker than Brave or Mullvad; UI layer is proprietary, not fully open-source; heavier RAM use |
| **Mullvad Browser** | Gecko (Tor Browser fork, no onion routing) | Tor-level anti-fingerprinting at normal speed; identical fingerprint across all users by design; no telemetry; built by Mullvad + Tor Project | No sync, no persistent bookmarks by default; needs a VPN for full benefit; small extension ecosystem; newer, less track record |
| **Tor Browser** | Gecko (heavily hardened) | Strongest anonymity available; routes through the Tor network; defeats fingerprinting and hides your IP/location | Slow — multi-hop routing; many sites break or block Tor exit nodes; unsuitable for daily browsing, streaming, banking |
| **Waterfox** | Gecko (Firefox fork) | Supports legacy XUL/XPCOM extensions Firefox dropped in 2017 | Niche use case; smaller maintainer base; not meaningfully more private than hardened Firefox |
| **Safari** | WebKit | Apple's Intelligent Tracking Prevention blocks cross-site cookies; iCloud Private Relay masks IP (paid tier) | Apple-ecosystem only; still sends data to Apple services; weaker anti-fingerprinting than Mullvad/Tor; closed-source |
| **Chrome / Edge** | Chromium | Best compatibility, DRM support | Built by ad companies; not a credible privacy choice regardless of settings |

**Why this guide uses Firefox:**
- It's the most practical middle point between privacy and compatibility for a daily driver.
- It uses an independently maintained engine, not a Chromium derivative.
- It stays fully compatible with mainstream sites, unlike Tor or Mullvad Browser.
- LibreWolf gets you most of this guide automatically — a legitimate shortcut if you don't want to configure things yourself.

**Where the others fit instead:**
- **Mullvad Browser or Tor Browser** — for genuinely high-risk or anonymous sessions, not daily browsing. Tor when you need to hide your IP/location outright; Mullvad Browser (paired with a VPN) when you want Tor-grade anti-fingerprinting at normal speed.
- **Brave or Vivaldi** — reasonable if you need Chrome extension compatibility and want privacy with near-zero configuration. Vivaldi trades some fingerprint resistance for far more built-in tooling.
- **Safari** — a sane default if you're Apple-only and don't want to install anything else, but not as hardened as the Firefox/Gecko options above.

> **Honesty note on Mozilla:** the 2025 Terms of Use update and default-on telemetry have drawn real criticism. This guide exists *because* Firefox isn't private out of the box — not to pretend otherwise.

---

## Part 2 — Locale & Language

What it is:
- `Accept-Language` is a header sent with every web request.
- It tells every site what language(s) your browser prefers.

Why it matters:
- A less common locale (e.g. German) narrows down who you are.
- English (`en-US`) is the most common configuration worldwide.
- Matching the majority reduces how much this header singles you out.
- This isn't about English being "better" — it's about blending into the largest group.

### Method 1: Change UI Language
1. Open `about:preferences#general`
2. Scroll to **Language**
3. Click **Set Alternative Languages…**
4. Remove your native locale, add `English (en-US)`
5. Restart Firefox

### Method 2: Fingerprinting Protection (Recommended)
```
about:config → privacy.resistFingerprinting = true
```
- Normalizes language automatically.
- Also forces UTC timezone and reduces entropy elsewhere.
- One setting, several fingerprinting vectors closed.

### Method 3: Set Accept-Language Directly
```
about:config → intl.accept_languages = en-US,en
```

**Verify:** test on `coveryourtracks.eff.org` to confirm what's actually being sent.

---

## Part 3 — Essential Hardening (High Value, Low Breakage)

### 1. Strict Tracking Protection

What it is:
- Firefox's built-in network-level blocker.

Where:
- `Settings → Privacy & Security → Enhanced Tracking Protection → Strict`

What it blocks:
- Cross-site trackers
- Cryptominers
- Fingerprinting scripts
- Tracking cookies

This is Firefox's equivalent of what Brave does by default.

---

### 2. Extensions: Less Is Measurably More

The intuition "more privacy tools = more privacy" is backwards. Here's the data behind that claim.

**Extension combinations are a fingerprint:**
- A large-scale study found 18.38% of users are uniquely identifiable purely from their specific combination of installed extensions.
- Among users with at least one detectable extension, 54.86% were unique because of it.
- Combined with logged-in website sessions: 34.51% of all users were unique.
- Among users with at least one extension *and* one login: 89.23% were unique.
- The detection technique runs in about 625 milliseconds — not slow, not theoretical.

**Why extensions are detectable at all:**
- Many extensions modify the page — inject buttons, alter the DOM, block specific elements.
- That modification is itself visible to a script running on the page.
- The exact *combination* of which modifications appear becomes a signature.

**Separately, baseline fingerprint uniqueness is already high:**
- EFF's large-scale research found 84% of ~500,000 browsers tested had a fully unique configuration — before counting extensions at all.
- That rises to 94% for browsers with older plugins like Flash or Java.

**Practical takeaway:**
- Every extension you add increases identifiability, not just functionality.
- Exception: extensions specifically engineered to suppress their own footprint, like uBlock Origin.
- Most extensions are not built with that goal in mind.

**On stacking multiple blockers (e.g. uBlock Origin, Privacy Badger, AdBlock Ultimate):**
- uBlock Origin (filter-list based) and Privacy Badger (EFF's behavioral tracker-learning tool) cover significantly overlapping ground on Firefox.
- On Firefox specifically, uBlock Origin alone already does most of the work that Privacy Badger adds.
- The overlap is large enough that running both produces marginal extra protection for most users.
- A third blocker on top (e.g. AdBlock Ultimate) adds almost no additional coverage.
- It does add: a third detectable extension fingerprint, a third codebase to trust, and — for some "free" ad blockers that monetize via paid whitelisting — a weaker trust profile than EFF/community-vetted tools.

**Recommended minimal stack:**
- **uBlock Origin** — ads, trackers, malware domains, cosmetic filtering. Does most of the work alone.
- **Firefox Multi-Account Containers** — not a blocker, but identity isolation (see Part 5). Matters more than any blocker.
- *(Optional, pick one, not both)* Privacy Badger **or** ClearURLs — behavioral tracking detection, or tracking-parameter stripping.

**Avoid:**
- Stacking 4–5+ "privacy/security" extensions
- Antivirus browser plugins
- Any extension whose source/permissions you haven't checked yourself

---

### 3. DNS-over-HTTPS (DoH)

What it is:
- Encrypts your DNS lookups so your ISP (or anyone on the network) can't see which domains you're resolving.

Where:
- `Settings → Privacy & Security → DNS over HTTPS`

Recommended providers:
- Cloudflare
- NextDNS
- Quad9

Why NextDNS stands out for security engineers:
- Lets you block telemetry, malware domains, typosquatting, and ad/tracker infrastructure at the resolver level.
- Gives full visibility into every query made.

---

### 4. Disable Telemetry

What telemetry is:
- Technical and interaction data Mozilla collects about how Firefox runs and how you use it.
- Includes: page load times, memory usage, which features you use, OS/browser version, hardware specs.
- Mozilla's stated purpose: bug fixing, performance tuning, feature prioritization.

Where:
- `about:preferences#privacy` → uncheck:
  - Technical data collection
  - Studies
  - Personalized suggestions
  - Advertising measurement

Why disable it anyway:
- The current stated use may be benign, but it's still a standing, automated data relationship with a third party.
- Independent reporting has found cases where "opt-out" didn't fully stop background pings.
- Mozilla's 2025 Terms of Use expanded what's collected for marketing purposes.
- Security principle: minimize standing data relationships you don't control, regardless of current intent.
- Cost of disabling it: a five-second toggle, effectively zero downside for daily use.

---

### 5. Force HTTPS

What it is:
- Automatically upgrades or blocks insecure `http://` connections.

Where:
- `Settings → Privacy & Security → HTTPS-Only Mode → Enable`

Why:
- Prevents accidental plaintext connections, e.g. from an old bookmark or typed URL.

---

## Part 4 — Advanced Hardening (`about:config`)

Type `about:config` in the address bar, accept the warning.

| Setting | Value | What it does | Why it matters |
|---|---|---|---|
| `privacy.resistFingerprinting` | `true` | Standardizes dozens of exposed properties at once (timezone, canvas output, font enumeration, screen resolution) | Single highest-impact setting here — makes your "shape" match a large pool of other users instead of standing out. **Can break some sites** (banking, streaming) — test first. |
| `media.peerconnection.enabled` | `false` | Disables WebRTC | WebRTC can leak your real local/public IP even through a VPN, by opening a direct peer connection that bypasses the tunnel. Critical for VPN/Tor use. |
| `network.prefetch-next` | `false` | Stops speculative pre-loading of predicted next pages | Prevents requests firing to URLs you never visited — a minor privacy leak and unnecessary traffic. |
| `network.http.speculative-parallel-limit` | `0` | Disables speculative DNS/TCP pre-connections | Closes a "phantom" network activity class invisible to you but visible to network monitors. |
| `dom.event.clipboardevents.enabled` | `false` | Disables clipboard event listeners | Stops sites detecting copy/paste — relevant to real clipboard-hijacking scams that swap copied URLs. |
| `geo.enabled` | `false` | Fully disables Geolocation API | Removes the risk of accidentally approving a location prompt. Re-enable only if you regularly use maps. |
| `dom.battery.enabled` | `false` | Disables Battery Status API | Legacy API that exposed exact battery % and charge rate — shown to help re-identify devices across sessions. |
| `beacon.enabled` | `false` | Disables Beacon API | Removes a silent "did the user leave the page" analytics channel that fires after navigation. |
| `webgl.disabled` | `true` *(optional)* | Fully disables WebGL | WebGL exposes your exact GPU model/driver — high-entropy fingerprint signal. **Will break** maps, 3D content, some CAPTCHAs. |
| `media.eme.enabled` | `false` *(optional)* | Disables Widevine DRM | DRM modules are closed-source binaries running inside your browser — real audit-blind attack surface. Breaks DRM streaming (Netflix, Spotify Web). |

> **On cookies:** Firefox's Strict mode (Part 3.1) already includes **Total Cookie Protection**, which isolates cookies per-site automatically. Manual `network.cookie.cookieBehavior` tuning is mostly unnecessary unless you want zero cross-session persistence (`= 5`), which logs you out of most sites on every restart.

---

## Part 5 — Compartmentalization

Why this matters more than any single setting:
- Config settings (Part 4) reduce what *one* site can learn about you.
- Compartmentalization controls what *different* sites can learn about each other by correlating your identity.

**Use Multi-Account Containers** — isolated cookie jars within one browser window:
- Personal
- Banking
- Cloud admin / infrastructure
- Research
- Social media
- Testing / disposable

Result:
- Drastically reduces cross-context tracking, cookie leakage, accidental auth bleed.
- One of Firefox's strongest structural advantages over Chromium browsers, which generally lack a native equivalent.

---

## Part 6 — Separate Profiles (Operational Security)

Difference from containers:
- Containers share the same browser instance.
- Profiles share nothing — no cookies, no extensions, no history, no session state.
- Profiles are a much stronger boundary.

Use separate profiles for:
- Personal life
- Work
- Offensive security research
- Malware analysis
- Anonymous / sensitive activity

**Launch a profile manager:**
```bash
firefox -P
```
Or navigate to:
```
about:profiles
```

### Sync — decide per profile, don't default to it

What Sync does:
- Stores history, saved passwords, open tabs, and bookmarks on Mozilla's servers.
- Encrypted, but still a standing copy of your data off your device.

Recommendation by profile type:
- **Personal/daily-driver:** Sync is reasonable if cross-device convenience matters to you and you trust Mozilla's encryption model.
- **Sensitive/research/anonymous:** leave Sync off entirely. No reason for this data to leave the local machine.

---

## Part 7 — Arkenfox (Advanced Baseline)

**[Arkenfox user.js](https://github.com/arkenfox/user.js)** — a maintained hardening config beyond manual tweaks.

What it does:
- Disables telemetry comprehensively, beyond the UI toggles in Part 3
- Hardens privacy across hundreds of preferences
- Improves fingerprint resistance with documented rationale per setting
- Explains *why*, not just *what* — worth reading even if not applied fully

Tradeoffs:
- Can break sites more aggressively than the moderate settings in Part 4
- Requires periodic maintenance as Firefox updates
- Best for users comfortable debugging breakage, not a "set and forget" daily driver

**Don't want to maintain this yourself?**
- LibreWolf (Part 1) ships Arkenfox-equivalent hardening by default, maintained by someone else.

---

## Anti-Patterns: What NOT to Do

- ❌ Stacking 4–5+ overlapping privacy/security extensions — the math in Part 3.2 works against you
- ❌ Running random GitHub "hardening scripts" without auditing them yourself
- ❌ Using "secure browsers" from tiny, unaudited teams with no public security disclosure process
- ❌ Disabling JavaScript globally for daily browsing — breaks most of the modern web, pushes you toward unsafe workarounds
- ❌ Applying Tor-Browser-style settings inside normal Firefox — Tor's defense works because every Tor user looks identical; the same tweaks alone in regular Firefox just make you the only one doing it
- ❌ Randomizing your user-agent — an inconsistent or rare user-agent is itself a strong fingerprinting signal

Net effect of these mistakes: higher fingerprintability, broken usability, and habits like routinely bypassing your own security warnings.

---

## Recommended Practical Setup

| Use Case | Configuration |
|---|---|
| **Daily Driver** | Firefox, Strict ETP, uBlock Origin only, Containers, Part 4 moderate hardening, Sync optional |
| **Research / Sensitive Work** | Dedicated profile, Sync off, or switch to Tor Browser for high-risk sessions |
| **"Set and forget" alternative** | LibreWolf as daily driver — most of this guide pre-applied |
| **Compatibility Fallback** | Brave or Chrome, minimal use only, for sites that break under strict settings |

---

## Key Takeaway

Largest real-world gains, in order of impact:

1. **Compartmentalization** (containers + profiles) — controls correlation across contexts
2. **Extension discipline** (one blocker, not four) — the math in Part 3.2 is unambiguous
3. **Identity isolation** (separate Sync decisions per profile)
4. **Consistent behavior** — not bypassing your own protections

Config settings in Part 4 matter, but they're the smallest lever on this list. A perfectly hardened `about:config` is undone the moment you:
- mix identities across contexts
- reuse sessions between sensitive and personal profiles
- install a fourth "just in case" extension
- click through your own warnings out of habit

Discipline makes configuration effective — not the other way around.
