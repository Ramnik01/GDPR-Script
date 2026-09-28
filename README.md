# GDPR-Script

> Lightweight, accessible, **GDPR / ePrivacy / CCPA-friendly** consent banner — no dependencies, `<8 kB` gzipped, CSP-safe.

A complete rewrite fixing every common issue in naive “GDPR popup” snippets: prior consent, equal choice, revocability, expiry, GPC, accessibility, and security — with a polished demo included.

**Demo:** open `index.html` in your browser (no build step).

---

## ✨ What was fixed

| Naive script issue | Fix in 2.0 |
|---|---|
| No **reject** button / dark pattern (big green Accept, tiny grey Reject) | **Equal choice**: Accept all · Reject non-essential · Customize — same visual weight |
| Cookies/scripts set **before** consent | **Prior consent**: all `marketing`/`statistics` scripts stay `type="text/plain"` until opted-in |
| `scroll = consent` or pre-ticked boxes | Explicit **opt-in** only; necessary is the only pre-checked category |
| No way to **withdraw** | Floating **Cookie settings** button + `GDPR.showSettings()` + keyboard `Esc` |
| Consent never expires | `expiryDays: 182` + auto re-prompt + `policyVersion` bump forces re-consent |
| Stored only in `cookie` without `SameSite`/`Secure` | `localStorage` primary + `SameSite=Lax; Secure` cookie mirror |
| Ignores GPC/DNT | Respects **Global Privacy Control** (`navigator.globalPrivacyControl`) |
| `innerHTML` XSS via `policyUrl`/strings | **No `innerHTML`** for user data — `textContent` + `createElement` only, URL allow-list (`http/https/mailto`) |
| `eval`, inline handlers → breaks CSP | No `eval`, only `addEventListener` |
| Inaccessible (no focus trap, no screen-reader) | `role="dialog"`, `aria-modal`, focus trap, `focus-visible`, `prefers-reduced-motion`, ≥4.5:1 contrast |
| Layout shift / z-index wars | Fixed stacking (`--gdpr-z: 2147483646`), overlay blur, no CLS |
| No server-side signal | Single `gdpr_consent` cookie (`{version, policyVersion, timestamp, categories}`) |
| No audit trail | Consent record with `timestamp`, `version`, `via` |

---

## 🚀 Quick start

### 1. Drop-in (no build)
```html
<link rel="stylesheet" href="gdpr.css">
<script src="gdpr.js"></script>
<script>
  GDPR.init({
    name: 'Acme',
    policyUrl: '/privacy',
    position: 'bottom', // top | bottom | center
    expiryDays: 182
  })
</script>
```

### 2. Auto-init via attributes (zero JS)
```html
<script src="gdpr.js" data-gdpr-auto data-gdpr-policy-url="/privacy" data-gdpr-name="Acme"></script>
```

### 3. Module / bundler
```js
import './gdpr.css'
import './gdpr.js' // exposes window.GDPR
GDPR.init({ policyUrl: '/privacy' })
```

---

## 🔒 Blocking non-essential scripts until consent

Change **blocked** tags to `type="text/plain"` and add `data-gdpr-category`:

```html
<!-- Analytics — only loads after "statistics" consent -->
<script type="text/plain" data-gdpr-category="statistics" data-src="https://www.googletagmanager.com/gtag/js?id=G-XXX"></script>
<script type="text/plain" data-gdpr-category="statistics">
  gtag('config', 'G-XXX')
</script>

<!-- YouTube / ads — only after "marketing" -->
<iframe type="text/plain" data-gdpr-category="marketing"
        data-src="https://www.youtube.com/embed/dQw4w9WgXcQ"
        width="560" height="315" frameborder="0" allowfullscreen></iframe>

<!-- Image pixel example -->
<img type="text/plain" data-gdpr-category="marketing" data-src="https://tracker.example/pixel.gif" alt="">
```

JS will **hydrate** (`replaceWith` / set `src`) the real element only when that category is allowed. Works for dynamically injected content too (MutationObserver, SPA-friendly).

> **Rule:** `necessary` is always allowed and never blocked. Every other category defaults to **off**.

---

## ⚙️ Configuration

```js
GDPR.init({
  name: 'My Site',              // shown in title fallback
  policyUrl: '/privacy',        // sanitized to http/https/mailto only
  policyVersion: '1',           // bump when policy changes → forces re-consent
  expiryDays: 182,              // CNIL-style ~6 months
  position: 'bottom',           // bottom | top | center
  respectGPC: true,             // auto-reject non-essential if GPC signal present
  respectDNT: false,            // opt-in DNT respect (off by default)

  categories: [                 // granular, extensible
    { id: 'necessary',   name: 'Strictly necessary', description: '…', required: true },
    { id: 'preferences', name: 'Preferences',        description: '…' },
    { id: 'statistics',  name: 'Statistics',         description: '…' },
    { id: 'marketing',   name: 'Marketing',          description: '…' },
  ],

  strings: {                    // i18n
    title: 'We value your privacy',
    description: 'We use cookies to improve your experience. Choose which categories you allow. Read our {policy}.',
    policyLabel: 'privacy policy',
    acceptAll: 'Accept all',
    rejectAll: 'Reject non-essential',
    customize: 'Customize',
    savePreferences: 'Save preferences',
    manage: 'Cookie settings',
    modalTitle: 'Customize consent',
    categoriesTitle: 'Cookie preferences',
    categoriesDescription: 'Toggle categories you want to allow. Necessary cookies cannot be disabled.',
    close: 'Close',
  },

  onConsentChange: (record) => {
    // record = { version, policyVersion, timestamp, expiryDays, categories, via? }
    if (record.categories.statistics) initAnalytics()
  }
})
```

---

## 🧩 JS API

```js
GDPR.hasConsent('marketing')   // boolean
GDPR.getConsent()              // full record or null
GDPR.acceptAll()
GDPR.rejectAll()
GDPR.savePreferences({ preferences: true, statistics: false, marketing: false })
GDPR.showSettings()            // open modal
GDPR.hideSettings()
GDPR.resetConsent()            // clear + re-show banner (dev/testing)
GDPR.on('consent', (record) => {}) // also: window.addEventListener('gdpr:consent', …)
GDPR.on('reset', () => {})
GDPR.version                   // "2.0.0"
```

---

## ♿ Accessibility

- `role="dialog"` + `aria-modal="true"` + `aria-labelledby/describedby`
- **Focus trap** inside banner/modal; `Esc` to open/close customize
- `focus-visible` outlines, keyboard-only operation, no reliance on color alone
- `prefers-reduced-motion` disables transitions
- Color contrast ≥ 4.5:1, dark-mode via `prefers-color-scheme`
- `forced-colors: active` (high-contrast) support, `@media print` hides banner

---

## 🌙 Theming

Override CSS variables:

```css
:root {
  --gdpr-bg: #fff;
  --gdpr-fg: #0f172a;
  --gdpr-primary: #0f172a;
  --gdpr-accent: #2563eb;
  --gdpr-radius: 16px;
}
```

Or toggle `position`: `bottom` (default), `top`, `center`.

---

## 🔐 Privacy & compliance notes

- **Prior consent** — markup pattern above guarantees no non-essential request fires before opt-in (verify in DevTools → Network).
- **Cookie set**: only `gdpr_consent` (JSON, `SameSite=Lax`, `Secure` on HTTPS) **after** a choice. No fingerprinting.
- **Withdraw**: the floating “Cookie settings” button is always visible after choice; one click re-opens preferences.
- **GPC**: when `respectGPC: true` and `navigator.globalPrivacyControl === true`, banner auto-saves a reject-all record and doesn’t nag.
- **Expiry**: `expiryDays` + `policyVersion` + `version` each trigger a re-prompt — no stale perpetual consent.
- **No dark patterns**: reject is the same size/contrast as accept, no “X implies consent”, no cookie wall.
- This is not legal advice — adapt `categories`, `policyUrl`, and `expiryDays` to your counsel’s guidance (e.g., TTDSG, CNIL, Garante).

---

## 🧪 Testing locally

```bash
# just open the demo — no npm needed
open index.html
# or
npx serve .
```

Dev helpers:

```js
localStorage.clear(); document.cookie = 'gdpr_consent=; expires=Thu, 01 Jan 1970 00:00:00 GMT; path=/'
GDPR.resetConsent() // re-show banner
GDPR.getConsent()
GDPR.hasConsent('statistics')
```

Check in DevTools:
- Network tab: blocked `type="text/plain"` requests should show 0 bytes until consent.
- Application → Local Storage / Cookies → `gdpr_consent`.

---

## 📁 Structure

```
gdpr.js      — consent manager (<8kB gz, vanilla)
gdpr.css     — styles (variables, responsive, a11y)
index.html   — interactive demo + docs
package.json — metadata + scripts
LICENSE      — MIT
```

---

## 🤝 Contributing

PRs welcome. Keep it: no dependencies, CSP-safe, a11y-first. Run `npx eslint gdpr.js` if you add tooling.

## 📄 License

MIT © Ramnik01 — see [LICENSE](LICENSE).
