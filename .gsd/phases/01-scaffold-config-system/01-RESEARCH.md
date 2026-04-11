# Phase 01: Scaffold & Config System — Research

**Researched:** 2026-04-11
**Domain:** Vanilla JS SPA, GitHub Pages static hosting, CSS custom properties, SPA view management
**Confidence:** HIGH — stack is straightforward web fundamentals; TRD provides detailed reference; verified against MDN

---

## Summary

This phase is pure web fundamentals: HTML/CSS/JS with no framework, hosted on GitHub Pages. The TRD (`../.github/TRD-v2-Workshop-Certificate-App.md`) already defines the full HTML structure, JS module organization, CSS patterns, and data schemas in detail. However, the TRD was written **before the discuss-phase**, meaning the CONTEXT.md locked decisions override several TRD choices — most critically the **config schema shape** (flat vs. nested) and the **CSS variable naming convention**.

The primary planning risk is the config schema conflict: the TRD uses nested `config.certificate.primary_color`; CONTEXT.md mandates flat `primary_color` at root level. The planner must use the flat schema and the JS functions that reference config values will differ from the TRD's reference code.

All other patterns (fetch + validation, view switching, image error handling, spinner) have correct reference implementations in the TRD — use them verbatim unless they conflict with CONTEXT.md.

**Primary recommendation:** Use the TRD as the structural reference, but substitute the flat config schema per CONTEXT.md decisions everywhere the TRD uses `config.certificate.*`.

---

<user_constraints>
## User Constraints (from CONTEXT.md)

### Locked Decisions

**Config Schema Fields**
- Schema fully defined upfront covering all phases — no incremental extension in later phases
- Flat naming convention: `org_name`, `logo_url`, `primary_color`, `secondary_color`
- Boolean feature flags at top level: `show_qr`, `show_description`, `show_seal` — all default `true`
- Certificate display fields: `certificate_title`, `issued_by_label`, `date_label`, `id_label`, `signature_name`, `signature_title`, `seal_url`, `signature_url`
- Search/landing page copy in config: headline, subtext, input placeholder, button label, footer note

**CSS Variable Coverage**
- Inject: `--primary-color`, `--secondary-color`, `--logo-url`, `--seal-url`, `--signature-url`
- Font families configurable: `--font-heading` and `--font-body`
- Certificate border: `--border-color` and `--border-width` from config
- CSS variables injected once at app boot (config load) — not re-injected on view transitions

**File & Module Organization**
- JavaScript: single `app.js` file
- CSS: single `styles.css` file — all styles including reset, variables, certificate, search, and error views
- Assets: `assets/images/` and `assets/fonts/` subdirectories
- Attendee data: `data/` at root level — alongside `config/` and `assets/`

**Error & Loading States**
- Loading indicator: pure CSS spinner
- Config failure error state: centered message — "Could not load configuration. Please try again."
- Error state styling: neutral grey/system colors — no branding applied
- Same error state for all failure types (network failure, JSON parse error, invalid config shape)

### Copilot's Discretion
- Specific spinner animation style/size
- Exact error icon (SVG inline or Unicode symbol)
- Internal JS function/variable naming conventions
- Search page copy field names in config (e.g., `search_headline` vs `landing_headline`)

### Deferred Ideas (OUT OF SCOPE)
None — discussion stayed within phase scope.
</user_constraints>

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| APP-03 | All views render within same `index.html` with no page reload | SPA view switching pattern via `.hidden` class toggle (§ Architecture Patterns) |
| APP-04 | User sees loading state while data is being fetched | Loading-view visible by default; spinner shows before any fetch runs (§ FOUC + Spinner) |
| CFG-01 | App fetches `config/certificate.config.json` on every page load before rendering anything | `init()` → `fetchConfig()` as first async action; no render before config resolves (§ Fetch Pattern) |
| CFG-02 | All visible text from config JSON — nothing hardcoded in HTML | Empty content-bearing elements in HTML; JS fills them via `.textContent` (§ HTML Structure) |
| CFG-03 | CSS color variables set dynamically from config JSON values | `document.documentElement.style.setProperty()` called after config loads (§ CSS Variable Injection) |
| CFG-04 | Config JSON drives all org settings, labels, colors, image URLs, feature flags | Full flat schema design covers all 6 phases (§ Complete Flat Config Schema) |
| DATA-01 | Each attendee has exactly one JSON file at `/data/{id}.json` | Convention documented in sample.json and PR template (§ sample.json Design) |
| DATA-02 | Filename follows email sanitization convention | `sanitizeEmail()` algorithm documented in TRD and research (§ Email Sanitization) |
| CONTRIB-01 | `/data/sample.json` exists as contributor template | Full sample.json content defined (§ sample.json Design) |
</phase_requirements>

---

## ⚠️ Critical Schema Conflict: TRD vs CONTEXT.md

This is the single most important finding for the planner.

**TRD defines a nested config schema** (`config.certificate.primary_color`, `config.certificate.show_qr`, etc.)  
**CONTEXT.md mandates a flat schema** (`config.primary_color`, `config.show_qr` directly at root)

**CONTEXT.md decisions are locked and take precedence.** The planner must use flat config everywhere.

**Impact on app.js:** Every reference to `config.certificate.*` in TRD's reference code must become `config.*` when implementing.

**CSS variable naming conflict:**
| TRD uses | CONTEXT.md mandates |
|----------|---------------------|
| `--cert-primary` | `--primary-color` |
| `--cert-border` | `--border-color` |
| `--cert-accent` | `--secondary-color` |
| `--cert-bg` | `--background-color` |
| `--cert-text` | (not specified — use `--text-color`) |
| `--cert-muted` | (not specified — use `--muted-color`) |

Use CONTEXT.md naming for all CSS variables. The `--cert-*` prefix used in TRD is superseded.

---

## Standard Stack

### Core (No Installation Required)

| Item | Version | Purpose | Rationale |
|------|---------|---------|-----------|
| HTML5 | — | Single `index.html` entry point | Zero build, works directly from GitHub Pages |
| CSS3 | — | Single `styles.css` with `@media print` | Full print media support, no preprocessor needed |
| Vanilla JS (ES6+) | — | Single `app.js` | No framework, no bundler, no `node_modules` |
| `fetch()` API | — | Config + data file fetching | Widely available, returns Promises, integrates with async/await |
| `URLSearchParams` | — | `?id=` param parsing | [VERIFIED: MDN] Baseline widely available, Chrome 49+, Safari 10.1+ |
| `document.documentElement.style.setProperty()` | — | CSS custom property injection | [VERIFIED: MDN] Direct inline style override on `:root` element |

### CDN Libraries (Loaded in index.html, not needed for Phase 01 logic)

| Library | Version | CDN | Used By |
|---------|---------|-----|---------|
| qrcode.js | 1.0.0 | cdnjs.cloudflare.com | Phase 03 |
| html2pdf.js | 0.10.1 | cdnjs.cloudflare.com | Phase 03 |
| Google Fonts (Playfair Display + Lato) | — | fonts.googleapis.com | Phase 02 |

> Phase 01 should include all CDN `<script>` tags and `<link>` tags in `index.html` even though the libraries are only used in later phases. This avoids HTML edits in Phase 03.

**CDN script tags (exact URLs from TRD §1.3):** [CITED: TRD v2 §1.3]
```html
<!-- Google Fonts -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,700;1,400&family=Lato:wght@300;400;700&display=swap" rel="stylesheet" />

<!-- QR Code -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"
        integrity="sha512-[get from cdnjs]" crossorigin="anonymous"></script>

<!-- html2pdf -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"
        integrity="sha512-[get from cdnjs]" crossorigin="anonymous"></script>
```
> ⚠️ Fetch SRI integrity hashes from https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js before writing — they're not in the TRD.

---

## Architecture Patterns

### Recommended Project Structure

```
{repo-root}/
├── index.html                     ← Single entry point, all views
├── robots.txt                     ← Phase 05 adds content, Phase 01 creates placeholder
├── config/
│   └── certificate.config.json    ← Full flat schema (all phases)
├── data/
│   └── sample.json               ← Contributor template
├── assets/
│   ├── app.js                    ← All JavaScript logic (single file)
│   ├── styles.css                ← All CSS incl. @media print (single file)
│   ├── images/                   ← org logo, seal, signature, og-image
│   └── fonts/                    ← Self-hosted fonts if needed (Phase 01 creates dir)
└── .github/
    └── PULL_REQUEST_TEMPLATE.md  ← Phase 06 populates this
```

> **TRD vs CONTEXT.md file path conflict:** TRD places CSS at `assets/css/style.css` and JS at `assets/js/app.js`. CONTEXT.md says single `styles.css` and single `app.js`. The simplest path consistent with both: `assets/styles.css` and `assets/app.js`. Use these paths in `index.html`'s `<link>` and `<script>` tags.

> **TRD image dir conflict:** TRD uses `assets/img/`. CONTEXT.md says `assets/images/`. Use `assets/images/`.

---

### Pattern 1: SPA View Switching via Hidden Class

**What:** All views exist in DOM at all times. JS toggles `.hidden` class to show/hide. No innerHTML replacement.

**Why:** Simpler than DOM injection; browser has all elements immediately; avoids re-querying the DOM.

**Critical detail:** `#loading-view` has **no `hidden` class in HTML**. All other views start with `class="view hidden"`. This means the spinner is visible as soon as the browser parses the HTML — before any JS executes. [CITED: TRD v2 §3.1]

```html
<!-- Loading: visible by default — NO hidden class -->
<div id="loading-view" class="view">
  <div class="loading-spinner"></div>
  <p>Loading...</p>
</div>

<!-- All others start hidden -->
<div id="search-view" class="view hidden"></div>
<div id="certificate-view" class="view hidden"></div>
<div id="error-view" class="view hidden"></div>
```

```javascript
// Source: TRD v2 §4.2 (adapted)
function showView(activeId) {
  const viewIds = ['loading-view', 'search-view', 'certificate-view', 'error-view'];
  viewIds.forEach(id => {
    const el = document.getElementById(id);
    if (el) el.classList.toggle('hidden', id !== activeId);
  });
}
```

```css
/* Source: TRD v2 §5 */
.view { width: 100%; }
.hidden { display: none !important; }
```

---

### Pattern 2: Fetch + JSON Parse + Shape Validation

**What:** fetch → check `res.ok` → `.json()` → validate required fields. All errors bubble to a single `try/catch` that shows the error view.

**Why this pattern:** Config failure, attendee 404, and invalid JSON shape all result in the same error view (locked decision), so unified error handling is correct. [CITED: TRD v2 §4.2]

```javascript
// Source: TRD v2 §4.2
async function fetchConfig() {
  const res = await fetch('config/certificate.config.json');
  if (!res.ok) throw new Error(`Config fetch failed: ${res.status}`);
  return res.json();
}

async function fetchAttendee(id) {
  const res = await fetch(`data/${id}.json`);
  if (!res.ok) throw new Error(`Attendee fetch failed: ${res.status}`);
  const data = await res.json();
  validateAttendee(data);
  return data;
}

function validateAttendee(data) {
  const required = ['certificate_id', 'name', 'email', 'workshop', 'date', 'date_iso'];
  for (const field of required) {
    if (!data[field]) throw new Error(`Missing required field: "${field}"`);
  }
}
```

**Config shape validation (Phase 01 addition):** Since config drives all rendering, add minimal required-field check on config too:

```javascript
function validateConfig(config) {
  const required = ['org_name', 'primary_color', 'certificate_title'];
  for (const field of required) {
    if (!config[field]) throw new Error(`Invalid config: missing "${field}"`);
  }
}
```

> Same-error-for-all-failures rule: the `catch` block calls `showError()` or `showConfigError()`. Since the locked decision is same error state, ALL failures go to the same centered error message ("Could not load configuration. Please try again." for config; "Certificate not found" or similar for attendee).

---

### Pattern 3: CSS Variable Injection at Boot

**What:** After config fetches successfully, set CSS custom properties on `:root` using inline styles. Done once before any view renders.

**FOUC prevention mechanism:** [VERIFIED: MDN CSS custom properties]
1. `index.html` is parsed → loading-view spinner is visible (no flash of unstyled content)
2. `styles.css` loads with fallback `:root` values → spinner renders with fallback colors
3. Config fetches → `applyConfigVars(config)` runs → `:root` inline style overrides fallbacks
4. `showView('search-view')` or `showView('certificate-view')` runs → content renders with correct brand colors

The spinner shows with fallback colors first (~50-100ms), then brand colors apply after config loads. This is acceptable and expected.

```javascript
// Source: adapted from TRD v2 §4.2, with CONTEXT.md CSS variable names
function applyConfigVars(config) {
  const root = document.documentElement.style;
  root.setProperty('--primary-color',    config.primary_color    || '#1a2e4a');
  root.setProperty('--secondary-color',  config.secondary_color  || '#c8a951');
  root.setProperty('--background-color', config.background_color || '#ffffff');
  root.setProperty('--text-color',       config.text_color       || '#333333');
  root.setProperty('--muted-color',      config.muted_color      || '#777777');
  root.setProperty('--border-color',     config.border_color     || '#c8a951');
  root.setProperty('--border-width',     config.border_width     || '7px');
  root.setProperty('--font-heading',     config.font_heading     || "'Playfair Display', Georgia, serif");
  root.setProperty('--font-body',        config.font_body        || "'Lato', 'Helvetica Neue', sans-serif");
  // Image URLs — set as CSS vars for potential CSS background-image use
  if (config.logo_url)      root.setProperty('--logo-url',      `url(${config.logo_url})`);
  if (config.seal_url)      root.setProperty('--seal-url',      `url(${config.seal_url})`);
  if (config.signature_url) root.setProperty('--signature-url', `url(${config.signature_url})`);
}
```

**CSS fallback defaults in `styles.css`:**
```css
/* Source: TRD v2 §5 adapted to CONTEXT.md variable names */
:root {
  --primary-color:    #1a2e4a;
  --secondary-color:  #c8a951;
  --background-color: #ffffff;
  --text-color:       #333333;
  --muted-color:      #777777;
  --border-color:     #c8a951;
  --border-width:     7px;
  --font-heading:     'Playfair Display', Georgia, serif;
  --font-body:        'Lato', 'Helvetica Neue', sans-serif;
}
```

---

### Pattern 4: URL Query Param Parsing

```javascript
// Source: MDN URLSearchParams
function getQueryParam(key) {
  return new URLSearchParams(window.location.search).get(key);
}

// Usage in init():
const rawId = getQueryParam('id');  // returns null if ?id= not present
```

**Edge cases handled by URLSearchParams:**
- `?id=` (empty value) → returns `''` (empty string, falsy — treated as no ID)
- `?id=jane-doe` → returns `'jane-doe'`
- URL-encoded values → automatically decoded (e.g., `%40` → `@`)
- Plus signs (`+`) decoded as spaces — important for email inputs. The `sanitizeId()` strips non-alphanumeric anyway.

---

### Pattern 5: Image 404 Graceful Handling

**What:** Set `onerror` handler on each image element that hides it on load failure. Certificate still renders completely. [CITED: TRD v2 §4.2]

```javascript
// Set programmatically after setting src attributes
['cert-logo', 'cert-seal', 'cert-signature'].forEach(id => {
  const el = document.getElementById(id);
  if (el) el.onerror = () => { el.style.display = 'none'; };
});
```

**Why `style.display = 'none'` and not `.hidden` class:** `onerror` fires synchronously when the image 404s. Setting inline style is simpler than managing class for this single case.

**CSS consideration:** Use `max-height` / `max-width` on images (not fixed `height` / `width`). When the image is hidden, no layout gap remains because `display: none` removes it from flow.

---

### Pattern 6: App Initialization Sequence

```javascript
// Source: TRD v2 §4.2
'use strict';

const CONFIG_PATH = 'config/certificate.config.json';
const DATA_PATH   = 'data/';

document.addEventListener('DOMContentLoaded', init);

async function init() {
  showView('loading-view');   // redundant but explicit — loading-view is default

  const rawId = getQueryParam('id');
  const id    = rawId ? sanitizeId(rawId) : null;

  try {
    if (id) {
      // Certificate mode: fetch config + attendee in parallel
      const [config, attendee] = await Promise.all([
        fetchConfig(),
        fetchAttendee(id)
      ]);
      applyConfigVars(config);
      renderCertificateView(config, attendee);
      showView('certificate-view');
    } else {
      // Search mode: config only
      const config = await fetchConfig();
      applyConfigVars(config);
      renderSearchView(config);
      showView('search-view');
    }
  } catch (err) {
    console.error('App error:', err);
    showError(id);
  }
}
```

> **Phase 01 scope:** The `renderCertificateView()` and `renderSearchView()` functions are stubs in Phase 01. Phase 02 fills the certificate render; Phase 04 fills the search render. Phase 01 just needs the skeleton with correct flow.

---

### Anti-Patterns to Avoid

- **Inner HTML injection:** All user-visible values go through `.textContent`, never `.innerHTML` — prevents XSS even for trusted config values. [CITED: TRD v2 §11 Security Model]
- **Hardcoded text in HTML:** `<h1>Certificate of Completion</h1>` → must be `<h1 id="cert-title"></h1>` populated by JS
- **Fetching before DOMContentLoaded:** Always start from `DOMContentLoaded` — accessing `getElementById` before DOM parses fails silently
- **Absolute URL paths for fetch:** Use relative paths (`'config/certificate.config.json'` not `'/config/certificate.config.json'`). Absolute paths from root break when the repo is not at the domain root (e.g., `username.github.io/repo-name/`)
- **Reopening file:// directly:** `fetch()` fails with CORS error when page is opened as `file://`. Must use a local server.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Query string parsing | Custom regex | `new URLSearchParams(window.location.search)` | Handles encoding, empty values, duplicates correctly |
| Fetch error detection | Check for null/undefined | `if (!res.ok) throw new Error(...)` | HTTP 4xx/5xx returns a response object, not a rejection |
| JSON parse errors | try/catch around JSON.parse | Let `.json()` throw naturally into outer catch | Unified error handling per locked decision |
| CSS variable override | Read/write stylesheet rules | `element.style.setProperty()` | Direct, synchronous, reliable |

---

## Complete Flat Config Schema

> **This is the authoritative schema for `config/certificate.config.json`.** It is flat (no nested objects) per CONTEXT.md locked decision, covers all 6 phases, and uses key names that mirror CSS variable names (underscores → hyphens convention).

```json
{
  "// ─── Org Identity ─────────────────────────────────────": "Phase 01+",
  "org_name":          "Acme Workshop Co.",
  "org_tagline":       "Learn. Build. Grow.",
  "org_website":       "https://example.com",
  "logo_url":          "assets/images/logo.png",

  "// ─── Colors ───────────────────────────────────────────": "Phase 01+",
  "primary_color":     "#1a2e4a",
  "secondary_color":   "#c8a951",
  "background_color":  "#ffffff",
  "text_color":        "#333333",
  "muted_color":       "#777777",

  "// ─── Border ────────────────────────────────────────────": "Phase 01+",
  "border_color":      "#c8a951",
  "border_width":      "7px",

  "// ─── Image URLs ────────────────────────────────────────": "Phase 01+",
  "seal_url":          "assets/images/seal.png",
  "signature_url":     "assets/images/signature.png",

  "// ─── Fonts ─────────────────────────────────────────────": "Phase 01+",
  "font_heading":      "'Playfair Display', Georgia, serif",
  "font_body":         "'Lato', 'Helvetica Neue', sans-serif",

  "// ─── Certificate Text Labels ────────────────────────────": "Phase 02+",
  "certificate_title": "Certificate of Completion",
  "pre_name_text":     "This is to certify that",
  "post_name_text":    "has successfully completed the workshop",
  "issued_by_label":   "Authorized By",
  "date_label":        "Date of Completion",
  "id_label":          "Certificate ID",
  "signature_name":    "Dr. Jane Smith",
  "signature_title":   "Workshop Director",

  "// ─── Feature Flags ─────────────────────────────────────": "Phase 02+",
  "show_qr":           true,
  "show_description":  true,
  "show_seal":         true,

  "// ─── Search Page Copy ──────────────────────────────────": "Phase 04+",
  "search_headline":    "Verify Your Certificate",
  "search_subtext":     "Enter the email address you used to register.",
  "search_placeholder": "your@email.com",
  "search_button":      "Find My Certificate",
  "search_footer_note": "Can't find yours? Contact us at hello@example.com",

  "// ─── PDF Settings ───────────────────────────────────────": "Phase 03+",
  "pdf_filename_prefix": "certificate",
  "pdf_format":          "a4",
  "pdf_orientation":     "landscape",
  "pdf_margin":          0,

  "// ─── SEO ─────────────────────────────────────────────────": "Phase 05+",
  "site_title":       "Workshop Certificates — Acme Workshop Co.",
  "og_image_url":     "assets/images/og-default.png",
  "twitter_handle":   "@acmeworkshops",

  "// ─── Template Meta ──────────────────────────────────────": "Internal",
  "template_version": "1.0.0"
}
```

> **Note on comment keys:** JSON does not support comments. The `"// ─── ... "` keys are documentation-only strings — acceptable since no JS code reads them and they don't conflict with real keys. Remove if you want strict JSON minimalism, but they add enormous readability value for organizers editing the file.

> **Naming convention note:** The CONTEXT.md says "search page copy field names at Copilot's discretion." The names chosen above (`search_headline`, `search_subtext`, `search_placeholder`, `search_button`, `search_footer_note`) are clean and self-documenting.

> **Fields that differ from TRD:** TRD's `org_logo` → `logo_url`. TRD's `certificate.border_color` → `border_color`. TRD's `certificate.heading_label` → `certificate_title`. TRD's `certificate.authorized_by` → `signature_name`. TRD's `pdf.filename_prefix` → `pdf_filename_prefix`. All flattened per CONTEXT.md.

---

## sample.json Design

> `/data/sample.json` — contributor template. Filename is the only exception to the email-to-ID convention; it's an explicit named template, not a real attendee.

```json
{
  "certificate_id": "jane-doe-at-example-com",
  "name":           "Jane Doe",
  "email":          "jane.doe@example.com",
  "workshop":       "Introduction to Machine Learning",
  "date":           "April 5, 2026",
  "date_iso":       "2026-04-05",
  "description":    "Completed 16 hours of hands-on training covering supervised learning, neural networks, and model evaluation."
}
```

**Field design rationale:**
- `certificate_id`: must exactly match the JSON filename (without `.json`) — reinforced in PR template
- `date`: human-readable free-form string (e.g., "April 5, 2026") — rendered on certificate
- `date_iso`: ISO 8601 format — used in `<time datetime="">` attribute and JSON-LD
- `description`: optional field; shown/hidden by `show_description` flag — include in sample so contributors know it exists
- Values use realistic-looking data, not placeholder strings like `"string"` — serves as documentation

**Email-to-ID algorithm** (used on filename, `certificate_id`, and search sanitization): [CITED: TRD v2 §10]
```javascript
function sanitizeEmail(email) {
  return email
    .toLowerCase()
    .trim()
    .replace(/\+/g, '-plus-')   // handle + aliases (rare)
    .replace(/@/g, '-at-')      // @ → -at-
    .replace(/\./g, '-')        // dots → hyphens
    .replace(/[^a-z0-9\-]/g, ''); // strip everything else
}
// jane.doe@example.com → jane-doe-at-example-com
// john+work@corp.io    → john-plus-work-at-corp-io
```

---

## CSS Spinner Implementation

> Copilot's discretion area — chose a reliable, layout-safe pattern.

```css
/* Loading spinner — pure CSS, no layout impact */
#loading-view {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  gap: 1rem;
  color: var(--muted-color, #777);
}

.loading-spinner {
  width: 40px;
  height: 40px;
  border: 3px solid #e0e0e0;
  border-top-color: var(--primary-color, #1a2e4a);
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
```

**Why this works before config loads:** `var(--primary-color, #1a2e4a)` has a fallback value (`#1a2e4a`). The spinner renders in fallback color immediately. When config loads and CSS var is set, a rotation later it picks up the brand color. Visually seamless at 0.8s cycle.

**No layout impact:** Spinner is inside `#loading-view` which takes `min-height: 100vh`. When `showView()` hides it via `.hidden { display: none !important }`, it fully leaves the document flow.

---

## GitHub Pages File Serving — Gotchas

[CITED: GitHub Pages docs — static site hosting, Linux file system]

### Same-Origin Fetch — No CORS Required
`index.html`, `config/certificate.config.json`, and `data/*.json` are all served from the same origin (`username.github.io`). The `fetch()` calls use relative URLs, so CORS headers are never needed. [ASSUMED — consistent with standard same-origin behavior]

### JSON Content-Type
GitHub Pages serves `.json` files with `Content-Type: application/json`. `res.json()` works without any extra configuration. [ASSUMED — standard static hosting behavior]

### Case Sensitivity (CRITICAL)
GitHub Pages runs on Linux. File paths are **case-sensitive**.
- `data/jane-doe-at-gmail-com.json` ≠ `data/Jane-Doe-at-Gmail-com.json`
- The email sanitization algorithm always lowercases → final IDs are always lowercase → file naming must always be lowercase
- **Consequence:** All attendee JSON files must be named in lowercase. The PR template must reinforce this and include the sanitization algorithm so contributors use it correctly.

### Relative Paths for Fetch (CRITICAL)
Use relative paths, not absolute paths from root:
```javascript
// CORRECT — works at username.github.io/repo-name/
fetch('config/certificate.config.json')

// WRONG — fails at username.github.io/repo-name/ (goes to root domain)
fetch('/config/certificate.config.json')
```
GitHub Pages for a project repo uses `username.github.io/repo-name/` as the base. Absolute paths starting with `/` bypass the repo subdirectory.

### 404 Behavior
When `data/{id}.json` doesn't exist, GitHub Pages returns its standard 404 page (not JSON). `res.ok` will be `false` (status 404), so `if (!res.ok) throw new Error(...)` fires correctly and goes to the error view. This is the correct behavior.

### Local Development (MUST use a server)
`file://` protocol causes `fetch()` to fail with CORS / network errors. Must use:
```bash
# Python 3 (installed on most systems)
python -m http.server 8000
# Then browse to http://localhost:8000
```
Or VS Code Live Server extension, or any local static server. This is a gotcha for contributors unfamiliar with web development. The README (Phase 06) should explain this clearly.

### Deploy Delay
After pushing to `main`, GitHub Pages takes 1-5 minutes to rebuild and deploy. Changes are NOT immediate. Don't test a push and then immediately expect to see it live.

### Jekyll Bypass (No `_config.yml`, no Jekyll)
GitHub Pages runs Jekyll by default. Jekyll ignores files/dirs starting with `_`. Since our project has no `_` prefixed directories and doesn't use liquid templates, Jekyll processes it transparently as a static site. Adding `.nojekyll` file (empty, at root) can be added as a safety measure to skip Jekyll processing entirely and speed up build. [ASSUMED — standard practice]

---

## Common Pitfalls

### Pitfall 1: FOUC During Config Load
**What goes wrong:** If the loading view ever has `hidden` class in HTML, or if JS removes `hidden` from a content view before config loads, unstyled content flashes.
**How to avoid:** `#loading-view` in HTML has NO `hidden` class. `showView('loading-view')` is the first line of `init()` (redundant but safe). `applyConfigVars(config)` runs BEFORE `showView()` for the final view. Order matters.

### Pitfall 2: fetch() Relative Paths Break on Subdirectory Hosting
**What goes wrong:** Path like `fetch('/config/certificate.config.json')` works locally but 404s on GitHub Pages because the app lives at `username.github.io/repo-name/`.
**How to avoid:** Always use relative paths without leading `/`.

### Pitfall 3: Config Comments Break JSON Parsing
**What goes wrong:** Adding `// Comment` lines to `certificate.config.json` for documentation causes `res.json()` to throw a parse error — the app shows the error state on every visit.
**How to avoid:** Use the `"// --- Section ---"` key approach shown in the flat schema above. Or use empty string values. Real JavaScript comments are never valid in JSON.

### Pitfall 4: View Content Visible Before JS Executes
**What goes wrong:** If any content-bearing HTML element in search/cert view has visible text (e.g., `<h1>Workshop Certificates</h1>`), it flashes before JS overwrites it.
**How to avoid:** All text-bearing elements in hidden views must be empty in HTML: `<h1 id="search-headline"></h1>`. Only the meta description and title tags in `<head>` can have placeholder text (they're not visible).

### Pitfall 5: Sanitize ID Before Building Fetch Path
**What goes wrong:** `fetch(\`data/${userInput}.json\`)` with unsanitized user input could be manipulated to fetch unexpected paths if the ID came from URL params.
**How to avoid:** `sanitizeId()` runs before fetch. It strips everything except `[a-z0-9-]` — path traversal (`../`) is impossible after sanitization. [CITED: TRD v2 §11]

### Pitfall 6: `onerror` on Image Must Be Set BEFORE Setting `src`
**What goes wrong:** Setting `img.onerror = handler` AFTER `img.src = url` may miss the error event in some browsers if the 404 response is cached (fires immediately).
**How to avoid:** Set the onerror handler before calling `setAttribute('src', ...)` or set it in HTML: `<img ... onerror="this.style.display='none'">`. In the TRD reference approach, it's set programmatically right after all src attributes are set — this works in practice because the fetch hasn't started yet for programmatic setting.

### Pitfall 7: `border_width` Requires CSS Unit
**What goes wrong:** Config value `7` (number) fails as a CSS property value. `document.documentElement.style.setProperty('--border-width', 7)` silently does nothing.
**How to avoid:** Store as a string in config: `"border_width": "7px"`. Document this convention in the config JSON comments.

---

## Code Examples

### Complete `init()` boot sequence
```javascript
// Source: TRD v2 §4.2 adapted for flat config + CONTEXT.md vars
'use strict';

const CONFIG_PATH = 'config/certificate.config.json';
const DATA_PATH   = 'data/';

document.addEventListener('DOMContentLoaded', init);

async function init() {
  showView('loading-view');
  const rawId = getQueryParam('id');
  const id    = rawId ? sanitizeId(rawId) : null;

  try {
    if (id) {
      const [config, attendee] = await Promise.all([
        fetchConfig(),
        fetchAttendee(id)
      ]);
      applyConfigVars(config);
      // Phase 02 fills renderCertificateView
      showView('certificate-view');
    } else {
      const config = await fetchConfig();
      applyConfigVars(config);
      // Phase 04 fills renderSearchView
      showView('search-view');
    }
  } catch (err) {
    console.error('[App] Error:', err);
    showConfigError(); // Single error state for all failures (CONTEXT.md locked)
  }
}

function showConfigError() {
  showView('error-view');
}
```

### Config-driven CSS variable injection (flat schema)
```javascript
function applyConfigVars(config) {
  const r = document.documentElement.style;
  r.setProperty('--primary-color',    config.primary_color    || '#1a2e4a');
  r.setProperty('--secondary-color',  config.secondary_color  || '#c8a951');
  r.setProperty('--background-color', config.background_color || '#ffffff');
  r.setProperty('--text-color',       config.text_color       || '#333333');
  r.setProperty('--muted-color',      config.muted_color      || '#777777');
  r.setProperty('--border-color',     config.border_color     || '#c8a951');
  r.setProperty('--border-width',     config.border_width     || '7px');
  r.setProperty('--font-heading',     config.font_heading     || "'Playfair Display', Georgia, serif");
  r.setProperty('--font-body',        config.font_body        || "'Lato', 'Helvetica Neue', sans-serif");
  if (config.logo_url)      r.setProperty('--logo-url',      `url(${config.logo_url})`);
  if (config.seal_url)      r.setProperty('--seal-url',      `url(${config.seal_url})`);
  if (config.signature_url) r.setProperty('--signature-url', `url(${config.signature_url})`);
}
```

### `showView()` function
```javascript
function showView(activeId) {
  ['loading-view', 'search-view', 'certificate-view', 'error-view'].forEach(id => {
    const el = document.getElementById(id);
    if (el) el.classList.toggle('hidden', id !== activeId);
  });
}
```

### URL param parsing and ID sanitization
```javascript
function getQueryParam(key) {
  return new URLSearchParams(window.location.search).get(key);
}

function sanitizeId(raw) {
  return raw.toLowerCase().replace(/[^a-z0-9\-]/g, '');
}

function sanitizeEmail(email) {
  return email.toLowerCase().trim()
    .replace(/\+/g, '-plus-')
    .replace(/@/g, '-at-')
    .replace(/\./g, '-')
    .replace(/[^a-z0-9\-]/g, '');
}
```

### `index.html` `<head>` structure
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />

  <!-- SEO base (JS overwrites these on certificate pages) -->
  <title>Workshop Certificates</title>
  <meta name="description" content="View and download your workshop completion certificate." />
  <link rel="canonical" href="" id="canonical-tag" />

  <!-- Open Graph (populated by JS) -->
  <meta property="og:type" content="website" />
  <meta property="og:title" content="" id="og-title" />
  <meta property="og:description" content="" id="og-description" />
  <meta property="og:url" content="" id="og-url" />
  <meta property="og:image" content="" id="og-image" />
  <meta property="og:site_name" content="" id="og-site-name" />

  <!-- Twitter Card (populated by JS) -->
  <meta name="twitter:card" content="summary_large_image" />
  <meta name="twitter:title" content="" id="tw-title" />
  <meta name="twitter:description" content="" id="tw-description" />
  <meta name="twitter:image" content="" id="tw-image" />
  <meta name="twitter:site" content="" id="tw-site" />

  <!-- JSON-LD placeholder (populated by JS in Phase 05) -->
  <script type="application/ld+json" id="json-ld-block">{}</script>

  <!-- Fonts -->
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,700;1,400&family=Lato:wght@300;400;700&display=swap" rel="stylesheet" />

  <!-- Styles -->
  <link rel="stylesheet" href="assets/styles.css" />
</head>
```

> Note: The `<link>` for `print.css` in TRD (`media="print"`) is collapsed into `styles.css` per CONTEXT.md's single-file decision. Use `@media print { }` block at the bottom of `styles.css`.

---

## State of the Art

| Old Approach | Current Approach | Notes |
|--------------|------------------|-------|
| `location.search` + custom regex | `new URLSearchParams(window.location.search)` | Built-in API since ES2019/broadly available |
| `classList.add/remove('hidden')` per element | `classList.toggle('hidden', condition)` | Single call, more readable |
| `style.cssText` for bulk CSS var setting | `style.setProperty()` per variable | More granular, allows reading back values |
| `document.write()` for dynamic content | `.textContent` via `getElementById` | Synchronous and safe |

---

## Environment Availability

> Step 2.6: Phase is purely code/config changes — single `index.html` static app, no external runtime dependencies beyond a browser. No build tools, no npm, no CLI utilities needed to create any of the source files. Python for local dev is available on most systems.

| Dependency | Required By | Available | Version | Fallback |
|------------|------------|-----------|---------|----------|
| Browser | All | ✓ | Any modern (ES6+) | — |
| Python 3 | Local dev server | ✓ (Windows/Mac/Linux) | 3.x | VS Code Live Server extension |
| Git | Committing work | ✓ | — | — |

No blocking dependencies.

---

## Validation Architecture

> `workflow.nyquist_validation` is not set in `.gsd/config.json` → treated as enabled.

This phase produces static files (HTML, JSON, CSS, JS) with no test framework in the repo yet. Validation is manual/browser-based for Phase 01. No test files need to be created in this phase.

### Phase 01 Verification Approach (Manual)

| Requirement | Verification |
|-------------|--------------|
| APP-03 (SPA no reload) | Open `index.html` via local server, change URL params — no full page reload |
| APP-04 (loading state) | Throttle network in DevTools, verify spinner appears |
| CFG-01 (config fetch first) | DevTools Network tab — config fetch appears before any render |
| CFG-02 (no hardcoded text) | View page source — no visible text strings in HTML |
| CFG-03 (CSS vars from config) | DevTools Elements → `:root` style — shows `--primary-color` set inline |
| CFG-04 (config drives all) | Change `primary_color` in config.json, refresh — UI updates |
| DATA-01, DATA-02 | sample.json file exists at correct path with correct naming |
| CONTRIB-01 | sample.json present, valid JSON, all required fields |

### Wave 0 Gaps
None — no automated test infrastructure needed for Phase 01 (pure static file creation).

---

## Security Domain

### Applicable ASVS Categories for Phase 01

| ASVS Category | Applies | Control |
|---------------|---------|---------|
| V5 Input Validation | YES — `?id=` param | `sanitizeId()` strips to `[a-z0-9-]` |
| V2 Authentication | No | Static site, no auth |
| V3 Session Management | No | No sessions |
| V4 Access Control | No | All files are public by design |
| V6 Cryptography | No | No crypto in Phase 01 |

### Threat Patterns for Vanilla JS Static App

| Pattern | STRIDE | Mitigation |
|---------|--------|------------|
| Path traversal via `?id=` | Tampering | `sanitizeId()` makes `../` impossible — only `[a-z0-9-]` allowed |
| XSS via config JSON values | Tampering | Use `.textContent` never `.innerHTML` for all config/attendee values |
| Open redirect via search input | Tampering | `sanitizeEmail()` and `sanitizeId()` strip all URL-special chars before building `?id=` redirect |
| JSON injection via data files | Tampering | Server-side only concern for this static app — JSON is parsed, not eval'd |

---

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | GitHub Pages serves `.json` files with `Content-Type: application/json` | GitHub Pages Gotchas | `res.json()` would fail; workaround: parse manually |
| A2 | `file://` fetch fails with CORS | GitHub Pages Gotchas | Local dev would work without a server — no fix needed |
| A3 | `onerror` on `<img>` fires reliably for 404s | Image Handling | Certificate might show broken icon; use CSS fallback if needed |
| A4 | `.nojekyll` file at root skips Jekyll processing | GitHub Pages Gotchas | Minimal risk — project has no `_` folders anyway |

---

## Open Questions

1. **CSS variable precedence with Google Fonts CDN failure**
   - What we know: If Google Fonts CDN is unavailable, `--font-heading` and `--font-body` CSS vars still reference the font family names, and browsers fall back to the system fonts in the value string (`Georgia, serif`)
   - What's unclear: Phase 01 doesn't load any fonts itself — fonts come from CDN link tag, already in `<head>`
   - Recommendation: Include system font fallbacks in `font_heading`/`font_body` config values and CSS var defaults. No action needed in Phase 01.

2. **`border_width` format for double border**
   - What we know: TRD CSS uses `border: 7px double var(--cert-border)`. The CSS property `border: {width} {style} {color}` takes entire shorthand.
   - What's unclear: If we store only `border_width: "7px"` in config, the double style is hardcoded in CSS. CONTEXT.md says "both `border_color` and `border_width` come from config" but doesn't mention border style.
   - Recommendation: Store `border_width: "7px"` and keep `double` as hardcoded CSS style. Document this in config comments. The planner can add `border_style` later if needed.

---

## Sources

### Primary (HIGH confidence)
- [MDN: URLSearchParams](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) — query param parsing API
- [MDN: Using CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascading_variables/Using_custom_properties) — CSS variable JS API
- [TRD v2](../.github/TRD-v2-Workshop-Certificate-App.md) — complete reference implementation for HTML structure, JS patterns, CSS

### Secondary (MEDIUM confidence)
- [GitHub Pages docs](https://docs.github.com/en/pages) — static hosting, deployment behavior
- CONTEXT.md (this repo) — locked design decisions from discuss-phase

### Tertiary (ASSUMED)
- GitHub Pages case sensitivity and JSON content-type behavior (A1, A2 in assumptions log)

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — pure web fundamentals, MDN-verified APIs
- Architecture patterns: HIGH — TRD provides detailed reference + MDN verification
- Config schema: HIGH — designed from CONTEXT.md locked decisions + all-phases review
- GitHub Pages gotchas: MEDIUM — standard static hosting behavior, A1/A2 assumed
- Pitfalls: HIGH — derived from code review of TRD patterns + known JS gotchas

**Research date:** 2026-04-11
**Valid until:** 2027-04-11 (web fundamentals are stable)
