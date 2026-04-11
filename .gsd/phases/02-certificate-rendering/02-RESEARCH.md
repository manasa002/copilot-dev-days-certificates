# Phase 02: Certificate Rendering — Research

**Researched:** 2026-04-11
**Domain:** Vanilla JS DOM manipulation, CSS A4 layout, fetch API parallel loading, graceful image error handling
**Confidence:** HIGH — pure web fundamentals; TRD v2 provides full reference implementation; conflicts with codebase are identified and documented

---

## Summary

Phase 02 fills in the certificate rendering subsystem — the core visual product of the app. The TRD (`/.github/TRD-v2-Workshop-Certificate-App.md`) provides a near-complete reference implementation. However, **the TRD was authored pre-discuss-phase and contains three systematic conflicts** with the locked codebase from Phase 01:

1. **Config schema shape**: TRD uses nested `config.certificate.*` access. The actual config is **flat** (`primary_color`, `seal_url`, `certificate_title` at root level).
2. **CSS variable names**: TRD uses `--cert-primary`, `--cert-border`, `--cert-accent`. The actual codebase uses `--primary-color`, `--border-color`, `--secondary-color`.
3. **File layout**: TRD references `assets/css/style.css` + `assets/css/print.css`. Actual paths are `assets/styles.css` (single file, print styles go in `@media print`).

Beyond these conflicts, the implementation is straightforward DOM manipulation + fetch. The planner can use the TRD's HTML structure and JS logic as the canonical reference — but must apply the three substitutions above throughout.

**Primary recommendation:** Use TRD HTML structure verbatim. Remap all `config.certificate.*` references to flat config keys per the mapping table below. Remap all `--cert-*` CSS variables to `--*-color` variants. Merge print styles into `styles.css` under `@media print`.

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| APP-02 | `/?id={id}` → renders Certificate view | `fetchAttendee()` + `renderCertificateView()` wired in `init()` branch (§ JS Patterns) |
| APP-05 | Parallel data fetch via `Promise.all()` | `init()` restructure — id branch uses `Promise.all([fetchConfig(), fetchAttendee(id)])` (§ Parallel Fetch) |
| DATA-03 | Attendee data fetched from `data/{id}.json` | `fetchAttendee(id)` fetches `data/${id}.json` (§ fetchAttendee) |
| DATA-04 | Missing/invalid ID shows error state with attempted ID | `fetchAttendee` throws on non-200; `showError(id)` populates `#error-id-display` (§ Error State) |
| CERT-01 | org logo, seal, signature, attendee name, workshop, date, org name, certificate_id | `renderCertificateView()` maps all config + attendee fields to HTML slots (§ Field Mapping) |
| CERT-02 | Seal hidden (not broken icon) when missing | `onerror` handler on `#cert-seal` sets `display:none` (§ Graceful Image 404) |
| CERT-03 | Landscape A4 (297mm × 210mm) proportions | `aspect-ratio: 297/210` on `.certificate` + `@media print { @page { size: A4 landscape; } }` (§ A4 Layout) |
| CERT-04 | Playfair Display headings + Lato body | Already loaded in `<head>`; CSS uses `var(--font-heading)` / `var(--font-body)` already in `:root` (§ Fonts) |
| CERT-05 | Description shown/hidden via `show_description` config flag | JS checks `config.show_description && attendee.description` before setting visibility (§ Field Conditionals) |
| CERT-06 | Certificate ID as small muted text at bottom | HTML slot `#cert-id-label` in cert-bottom-row; populated with `config.id_label + attendee.certificate_id` (§ HTML Structure) |
| CERT-07 | Border uses `border_color` / `border_width` from config | CSS: `border: var(--border-width) double var(--border-color)` — vars already set by `applyConfigVars()` in Phase 01 (§ CSS Variables) |
</phase_requirements>

---

## Standard Stack

### Core
| Technology | Version | Purpose | Why Standard |
|------------|---------|---------|--------------|
| Vanilla JS ES6+ | — | DOM manipulation, fetch, Promise.all | Locked constraint: no framework |
| CSS Custom Properties | — | Dynamic theming from config | Already established in Phase 01 |
| CSS `aspect-ratio` | — | A4 proportions on screen | 97%+ browser support [VERIFIED: MDN, 2025] |
| `@media print` + `@page` | — | Print/PDF layout control | Standard CSS — no library needed [VERIFIED: MDN] |
| Fetch API | — | JSON data loading | Browser-native, no polyfill needed for GitHub Pages |

### No Additional Dependencies
Phase 02 introduces zero new CDN libraries. All capabilities are browser-native.

---

## Architecture Patterns

### Plan Boundary Map

| Plan | File(s) Changed | What Gets Added |
|------|-----------------|-----------------|
| 02-01 certificate-structure | `index.html` only | Certificate HTML markup inside `#certificate-view`; update `#error-view` for dynamic ID display |
| 02-02 certificate-layout | `assets/styles.css` only | All certificate card CSS + `@media print` block |
| 02-03 certificate-data | `assets/app.js` only | `fetchAttendee()`, `validateAttendee()`, restructure `init()` for `Promise.all()` |
| 02-04 certificate-render | `assets/app.js` only | `renderCertificateView()`, `showError(id)` update, image `onerror` wiring |

### Recommended HTML Structure (certificate-view)

Add inside `<div id="certificate-view" class="view hidden">` in `index.html`. No JS injection — static HTML populated by JS via `textContent`/`setAttribute`.

```html
<div id="certificate-view" class="view hidden">
  <!-- Certificate Card -->
  <div class="certificate-wrapper">
    <div class="certificate" id="certificate">

      <!-- Top: Org Header -->
      <div class="cert-header">
        <img id="cert-logo" src="" alt="" class="cert-logo" />
        <p class="cert-org-name" id="cert-org-name"></p>
      </div>

      <!-- Decorative divider + title -->
      <div class="cert-title-block">
        <div class="cert-ornament-line"></div>
        <p class="cert-heading-label" id="cert-heading-label"></p>
        <div class="cert-ornament-line"></div>
      </div>

      <!-- Body: Name + Workshop + Description -->
      <div class="cert-body">
        <p class="cert-pre-name-text" id="cert-pre-name-text"></p>
        <h1 class="cert-name" id="cert-name"></h1>
        <p class="cert-post-name-text" id="cert-post-name-text"></p>
        <h2 class="cert-workshop" id="cert-workshop"></h2>
        <p class="cert-description hidden" id="cert-description"></p>
      </div>

      <!-- Footer: Date | Seal | Signature -->
      <div class="cert-footer">
        <div class="cert-footer-left">
          <time class="cert-date" id="cert-date" datetime=""></time>
          <span class="cert-footer-label" id="cert-date-label"></span>
        </div>
        <div class="cert-footer-center">
          <img id="cert-seal" src="" alt="Official Seal" class="cert-seal" />
        </div>
        <div class="cert-footer-right">
          <img id="cert-signature" src="" alt="Authorized Signature" class="cert-signature" />
          <p class="cert-authorized-name" id="cert-authorized-name"></p>
          <span class="cert-footer-label" id="cert-sig-label"></span>
        </div>
      </div>

      <!-- Bottom: Certificate ID -->
      <div class="cert-bottom-row">
        <p class="cert-id-label" id="cert-id-label"></p>
      </div>

    </div>
  </div>
</div>
```

> **Note:** QR code container (`#cert-qr`) is Phase 03. Omit from Phase 02. Action bar ("Download PDF", "Print") is also Phase 03.

### Error View Update (index.html)

The current `#error-view` has a hardcoded message "Could not load configuration." Phase 02 needs it to also display the attempted certificate ID. Update the error-view to support dynamic content:

```html
<div id="error-view" class="view hidden" role="alert">
  <div class="error-container">
    <span class="error-icon" aria-hidden="true">⚠</span>
    <p class="error-message" id="error-message">Could not load configuration. Please try again.</p>
    <p class="error-detail hidden" id="error-detail"></p>
  </div>
</div>
```

`showError(id)` sets `#error-message` to "Certificate not found." and populates `#error-detail` with the attempted ID. Config failures leave both as their default values (no `showError()` call — existing `catch` still shows error-view, but without ID detail).

**Why this preserves Phase 01 decisions:** Same visual component, same neutral styling. "Same error state for all failure types" is satisfied — it's one view, same CSS. The content is dynamic, not the design.

### Parallel Fetch Pattern (init() Restructure)

The current `init()` always fetches config first, then conditionally handles the id case. Phase 02 restructures to use `Promise.all()` when id is present [VERIFIED: MDN Fetch API docs]:

```javascript
async function init() {
  showView('loading-view');
  const rawId = getQueryParam('id');
  const id = rawId ? sanitizeId(rawId) : null;

  try {
    if (id) {
      // CERT mode: fetch both in parallel
      const [config, attendee] = await Promise.all([
        fetchConfig(),
        fetchAttendee(id)
      ]);
      validateConfig(config);
      applyConfigVars(config);
      if (config.site_title) document.title = config.site_title;
      renderCertificateView(config, attendee);
      showView('certificate-view');
    } else {
      // SEARCH mode: config only (Phase 04 will renderSearchView)
      const config = await fetchConfig();
      validateConfig(config);
      applyConfigVars(config);
      if (config.site_title) document.title = config.site_title;
      showView('search-view');
    }
  } catch (err) {
    console.error('[App] init error:', err);
    showError(id);
  }
}
```

> **Key change**: `showError(id)` (new) replaces bare `showView('error-view')` in the catch block. It populates `#error-message` contextually and always transitions to error-view.

### fetchAttendee Pattern

```javascript
async function fetchAttendee(id) {
  const res = await fetch('data/' + id + '.json');
  if (!res.ok) throw new Error('Attendee not found: ' + id + ' (HTTP ' + res.status + ')');
  const data = await res.json();
  validateAttendee(data);
  return data;
}

function validateAttendee(data) {
  var required = ['certificate_id', 'name', 'email', 'workshop', 'date', 'date_iso'];
  for (var i = 0; i < required.length; i++) {
    if (!data[required[i]]) throw new Error('Missing required field: ' + required[i]);
  }
}
```

### renderCertificateView: Config Key Mapping

This is the most critical reference for the planner. The TRD's `renderCertificateView` uses nested paths; the real config uses flat keys:

| HTML Element ID | Config key (FLAT — use this) | TRD key (WRONG — do not use) |
|----------------|------------------------------|-------------------------------|
| `#cert-heading-label` | `config.certificate_title` | `config.certificate.heading_label` |
| `#cert-pre-name-text` | `config.pre_name_text` | `config.certificate.pre_name_text` |
| `#cert-post-name-text` | `config.post_name_text` | `config.certificate.post_name_text` |
| `#cert-date-label` | `config.date_label` | `config.certificate.footer_left_label` |
| `#cert-sig-label` | `config.issued_by_label` | `config.certificate.footer_right_label` |
| `#cert-authorized-name` | `config.signature_name` | `config.certificate.authorized_by` |
| `#cert-org-name` (textContent) | `config.org_name` | `config.org_name` ✓ (same) |
| `#cert-logo` src | `config.logo_url` | `config.org_logo` |
| `#cert-seal` src | `config.seal_url` | `config.certificate.seal_image` |
| `#cert-signature` src | `config.signature_url` | `config.certificate.signature_image` |
| `#cert-description` visibility | `config.show_description` | `config.certificate.show_description` |
| `#cert-id-label` | `config.id_label + ': ' + attendee.certificate_id` | TRD uses `"Certificate ID: " hardcoded` |

Attendee fields are identical between TRD and actual schema: `attendee.name`, `attendee.workshop`, `attendee.date`, `attendee.date_iso`, `attendee.certificate_id`, `attendee.description`.

### renderCertificateView Reference Implementation

```javascript
function renderCertificateView(config, attendee) {
  // Page title
  document.title = attendee.name + ' — ' + attendee.workshop + ' | ' + config.org_name;

  // Org header
  var logo = document.getElementById('cert-logo');
  if (logo) { logo.src = config.logo_url || ''; logo.alt = config.org_name || ''; }
  setText('cert-org-name', config.org_name);

  // Labels from flat config
  setText('cert-heading-label',  config.certificate_title);
  setText('cert-pre-name-text',  config.pre_name_text);
  setText('cert-post-name-text', config.post_name_text);
  setText('cert-date-label',     config.date_label);
  setText('cert-sig-label',      config.issued_by_label);
  setText('cert-authorized-name', config.signature_name);

  // Attendee data
  setText('cert-name',     attendee.name);
  setText('cert-workshop', attendee.workshop);

  var dateEl = document.getElementById('cert-date');
  if (dateEl) {
    dateEl.textContent = attendee.date;
    dateEl.setAttribute('datetime', attendee.date_iso);
  }

  // Description: shown only if config flag + data both present
  var descEl = document.getElementById('cert-description');
  if (descEl) {
    var showDesc = config.show_description && attendee.description;
    if (showDesc) {
      descEl.textContent = attendee.description;
      descEl.classList.remove('hidden');
    } else {
      descEl.classList.add('hidden');
    }
  }

  // Certificate ID line
  setText('cert-id-label', (config.id_label || 'Certificate ID') + ': ' + attendee.certificate_id);

  // Images with graceful 404 handling
  setImageGraceful('cert-logo',      config.logo_url);
  setImageGraceful('cert-seal',      config.show_seal !== false ? config.seal_url : null);
  setImageGraceful('cert-signature', config.signature_url);
}

function setImageGraceful(id, src) {
  var el = document.getElementById(id);
  if (!el) return;
  if (!src) { el.style.display = 'none'; return; }
  el.onerror = function() { this.style.display = 'none'; };
  el.src = src;
}
```

> **`setImageGraceful` helper**: Sets `onerror` BEFORE `src` to ensure the event fires even if the image loads synchronously from cache in some browsers. Setting src then onerror can miss the error event. [VERIFIED: MDN HTMLElement.onerror, browser behavior docs]

### showError Update

```javascript
function showError(id) {
  var msgEl = document.getElementById('error-message');
  var detailEl = document.getElementById('error-detail');
  if (id && msgEl) {
    msgEl.textContent = 'Certificate not found.';
    if (detailEl) {
      detailEl.textContent = 'We could not find a certificate for: ' + id;
      detailEl.classList.remove('hidden');
    }
  }
  showView('error-view');
}
```

---

## A4 Layout

### Screen Display (CSS)

The certificate must appear landscape A4 proportioned on screen without requiring exact mm rendering. Two approaches:

| Approach | Implementation | Tradeoff |
|----------|----------------|----------|
| `aspect-ratio: 297/210` | `aspect-ratio: 297/210; width: 100%; max-width: 1050px` | Clean, responsive, correct ratio always maintained [VERIFIED: MDN] |
| Fixed pixel approximation | `min-height: 560px; max-width: 1050px` | TRD approach — does not maintain exact ratio on all screen widths |

**Recommendation:** Use `aspect-ratio: 297/210` — it's supported in all modern browsers and gives the exact A4 ratio without math. [VERIFIED: MDN, caniuse.com — 97%+ global support as of 2025]

```css
.certificate-wrapper {
  width: 100%;
  max-width: 1050px;
  margin: 2rem auto;
  padding: 0 1.5rem;
}

.certificate {
  background: var(--background-color);
  border: var(--border-width) double var(--border-color);
  padding: 48px 64px;
  box-shadow: 0 12px 48px rgba(0,0,0,0.12);
  display: flex;
  flex-direction: column;
  gap: 0;
  text-align: center;
  position: relative;
  width: 100%;
  aspect-ratio: 297 / 210;
  overflow: hidden;
}
```

### Print/PDF (CSS — in styles.css under @media print)

Phase 01 locked **single `styles.css`** — no separate `print.css`. Print styles go in a `@media print` block:

```css
@media print {
  @page {
    size: A4 landscape;
    margin: 0;
  }

  body {
    background: white !important;
  }

  .no-print,
  #search-view,
  #error-view,
  #loading-view {
    display: none !important;
  }

  #certificate-view {
    display: block !important;
  }

  .certificate-wrapper {
    max-width: 100%;
    padding: 0;
    margin: 0;
  }

  .certificate {
    box-shadow: none !important;
    width: 297mm;
    height: 210mm;
    aspect-ratio: auto; /* override screen ratio — use exact mm in print */
    padding: 14mm 20mm;
    page-break-inside: avoid;
    overflow: hidden;
    print-color-adjust: exact;
    -webkit-print-color-adjust: exact;
  }

  * {
    print-color-adjust: exact;
    -webkit-print-color-adjust: exact;
  }
}
```

> **Why `aspect-ratio: auto` in print**: The screen rule sets `aspect-ratio: 297/210` which would conflict with the explicit `width: 297mm; height: 210mm` in print. Override it to `auto` so the explicit mm values take full control. [ASSUMED — based on CSS cascade behavior; verify during implementation if any rendering issue]

---

## CSS Variable Mapping

Phase 01 established these CSS variable names (already in `:root` defaults in `styles.css`). Phase 02 must NOT use the TRD's `--cert-*` names — they do not exist:

| TRD variable (DO NOT USE) | Actual variable (USE THIS) | Set by |
|---------------------------|---------------------------|--------|
| `--cert-primary` | `--primary-color` | `applyConfigVars()` |
| `--cert-border` | `--border-color` | `applyConfigVars()` |
| `--cert-accent` | `--secondary-color` | `applyConfigVars()` |
| `--cert-bg` | `--background-color` | `applyConfigVars()` |
| `--cert-text` | `--text-color` | `applyConfigVars()` |
| `--cert-muted` | `--muted-color` | `applyConfigVars()` |
| `--font-heading` | `--font-heading` | `applyConfigVars()` ✓ same |
| `--font-body` | `--font-body` | `applyConfigVars()` ✓ same |

The `--border-width` variable is also set by `applyConfigVars()` from `config.border_width`.

**Important implication**: `.certificate` border should be:
```css
border: var(--border-width) double var(--border-color);
```

Not `7px double var(--border-color)` — the width must be the variable so config changes propagate.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| A4 proportions | Custom aspect-ratio math | CSS `aspect-ratio: 297/210` | Browser-native, responsive, no JS needed |
| Print page sizing | JS dimension calculation | `@page { size: A4 landscape; }` CSS rule | CSS spec handles this — JS has no access to print page dimensions |
| Font loading | Self-hosted fonts | Google Fonts CDN (already in `<head>`) | Already loaded in Phase 01; adding again would cause double-load |
| Image error handling | Retry logic or placeholder images | `el.onerror = () => el.style.display = 'none'` | One-liner; retry adds complexity for a graceful degradation case |

---

## Fonts

Google Fonts for Playfair Display and Lato are **already loaded** in `index.html` from Phase 01:

```html
<link href="https://fonts.googleapis.com/css2?family=Playfair+Display:ital,wght@0,400;0,700;1,400&family=Lato:wght@300;400;700&display=swap" rel="stylesheet">
```

CSS usage (already available in `:root` via `applyConfigVars()`):
```css
heading elements: font-family: var(--font-heading);   /* 'Playfair Display', Georgia, serif */
body/label elements: font-family: var(--font-body);   /* 'Lato', 'Helvetica Neue', sans-serif */
```

No Phase 02 work needed for font loading. Just apply the variables to certificate elements.

**Specific font usage on certificate:**
- `h1.cert-name`: Playfair Display, 700 weight, large — `var(--font-heading)`
- `h2.cert-workshop`: Playfair Display, italic 400 — `var(--font-heading); font-style: italic; font-weight: 400`  
- `p.cert-heading-label`: Playfair Display regular, spaced uppercase — `var(--font-heading)`
- All other text (pre/post name, date, labels, description, ID): Lato — `var(--font-body)` (inherits from body)

---

## Common Pitfalls

### Pitfall 1: TRD Nested Config References
**What goes wrong:** Planner/executor uses `config.certificate.primary_color` or `config.certificate.show_description` directly from TRD code — these are `undefined` with the flat schema.
**Why it happens:** TRD was written before discuss-phase locked the flat schema.
**How to avoid:** Always use the Field Mapping table above. Search for `config.certificate` in any written code — it should not exist.
**Warning signs:** If a field renders as blank/undefined but the config value is set, suspect nested access bug.

### Pitfall 2: Wrong CSS Variable Names
**What goes wrong:** Certificate appears with wrong colors — using `var(--cert-primary)` references an undefined variable, falling back to the browser default (usually black/transparent).
**Why it happens:** TRD CSS uses `--cert-*` names.
**How to avoid:** Phase 02 CSS must use `--primary-color`, `--border-color`, `--secondary-color`, `--muted-color`, `--text-color`, `--background-color`.

### Pitfall 3: Setting img.src Before img.onerror
**What goes wrong:** Image loads from browser cache faster than `onerror` handler is attached — 404 fires before the handler exists, broken icon appears.
**How to avoid:** Always set `onerror` BEFORE setting `src`. Use the `setImageGraceful()` helper pattern documented above.

### Pitfall 4: aspect-ratio Conflicts with Print
**What goes wrong:** Screen CSS `aspect-ratio: 297/210` overrides the explicit `width: 297mm; height: 210mm` in `@media print`, causing incorrect PDF/print dimensions.
**How to avoid:** Add `aspect-ratio: auto` to the `.certificate` rule inside `@media print` to neutralize the screen rule.

### Pitfall 5: Using innerHTML Instead of textContent
**What goes wrong:** XSS risk if any attendee.name or config field contains `<script>` or `<img onerror>` payloads.
**How to avoid:** All field population uses `.textContent`, never `.innerHTML`. The `setText()` helper from Phase 01 already uses `textContent`. [VERIFIED: existing app.js setText implementation]

### Pitfall 6: print.css as Separate File
**What goes wrong:** Phase 01 locked a single `styles.css`. Creating `assets/print.css` would require adding a `<link>` to `index.html` and contradicts the locked decision.
**How to avoid:** All print rules go in `@media print {}` block at bottom of `assets/styles.css`.

### Pitfall 7: Error View Without ID Display
**What goes wrong:** DATA-04 requires showing the attempted ID in the error state. If `error-view` HTML has no slot for the ID, `showError(id)` has nowhere to display it.
**How to avoid:** Plan 02-01 must update `#error-view` to add `#error-message` (with id, currently anonymous `<p>`) and `#error-detail` elements before Plan 02-04 writes the JS.

---

## Field Conditionals

The certificate has three conditional-render fields based on config flags:

| Field | Condition | Behavior |
|-------|-----------|----------|
| `#cert-description` | `config.show_description && attendee.description` | Show if BOTH flag is true AND data exists; hide otherwise |
| `#cert-seal` img | `config.show_seal !== false && config.seal_url` | Hide if flag is false OR url is empty; onerror hides if 404 |
| Any image | Image 404s | `onerror` → `style.display = 'none'` |

`show_seal` defaults to `true` in config. Pattern: `config.show_seal !== false` treats missing as true (backward-compatible).

---

## Environment Availability

Step 2.6: Environment audit applies only to external tools/services. This phase is purely browser-native code changes — no CLI tools, no databases, no external service calls beyond static file serving.

All external dependencies (Google Fonts CDN, GitHub Pages static hosting) were confirmed operational in Phase 01.

**SKIPPED — no external tools, services, or runtimes beyond what Phase 01 verified.**

---

## Validation Architecture

`nyquist_validation` is not set in `.gsd/config.json` — treating as enabled.

### Test Framework Reality

This project uses **pure vanilla JS with no build pipeline, no npm, no Node.js**. Standard automated test frameworks (Jest, Vitest, Playwright) require Node.js and npm to install — which are prohibited by the locked stack constraints.

**There is no viable automated test framework for this stack without violating Phase 01 constraints.** [ASSUMED — no project test config found; stack constraints prohibit Node-based runners]

### Phase Requirements → Validation Map

| Req ID | Behavior | Test Type | Method |
|--------|----------|-----------|--------|
| APP-02 | `/?id=sample-id` renders certificate | Manual browser verification | Open `index.html?id=jane-doe-at-example-com` | 
| APP-05 | Config + attendee fetch in parallel | Manual: Network tab in DevTools | Check waterfall — both requests start within 1 frame |
| DATA-03 | Attendee fetched from `data/{id}.json` | Manual: Network tab | Confirm request to `data/jane-doe-at-example-com.json` |
| DATA-04 | Invalid ID → error with attempted ID | Manual browser verification | Open `/?id=fake-id` — error must quote "fake-id" |
| CERT-01 | All fields populated | Manual visual check | Compare rendered cert against sample.json values |
| CERT-02 | Broken seal → no broken icon | Manual: Edit seal_url to 404 path | Flag absent, no broken img icon visible |
| CERT-03 | A4 landscape proportions | DevTools element ruler / print preview | Certificate wrapper ~1.41:1 width/height ratio |
| CERT-04 | Correct fonts applied | DevTools Computed tab | `font-family` resolves to Playfair Display / Lato |
| CERT-05 | show_description:false hides description | Manual: Set config flag | Description absent even with `data/sample.json` having one |
| CERT-06 | Certificate ID in muted text at bottom | Manual visual check | ID text visible at bottom of certificate |
| CERT-07 | Border from config | Manual: Change border_color in config | Certificate border color updates on reload |

### Wave 0 Gaps
None — no test files need to be created. Manual verification is the gate for this phase.

---

## Security Domain

### Applicable ASVS Categories

| ASVS Category | Applies | Control |
|---------------|---------|---------|
| V5 Input Validation | YES — `?id=` URL param | `sanitizeId()` from Phase 01 strips to `[a-z0-9\-]` only |
| V2 Authentication | NO | No auth; public certificate viewer |
| V4 Access Control | NO | All data is public static JSON |
| V6 Cryptography | NO | No secrets managed by app |
| V14 Configuration | PARTIAL | SRI hashes already on CDN scripts from Phase 01 |

### Threat Patterns

| Pattern | STRIDE | Mitigation |
|---------|--------|------------|
| Path traversal via `?id=` | Tampering | `sanitizeId()` restricts to `[a-z0-9\-]`, dots already stripped — `../` impossible |
| XSS via attendee name/workshop data | Tampering | All DOM writes use `.textContent` (not `.innerHTML`) — character entities not interpreted |
| CORS data fetch abuse | Information Disclosure | Fetches from same GitHub Pages origin — CORS not applicable |
| Prototype pollution via `config.show_seal !== false` check | Tampering | Config is plain deserialized JSON, not user-controlled; prototype pollution not a realistic vector for static config files |

No new security work required in Phase 02 — `sanitizeId()` and `textContent`-only rendering patterns from Phase 01 already cover the threat surface.

---

## Assumptions Log

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | `aspect-ratio: auto` inside `@media print` overrides the screen `aspect-ratio: 297/210` rule | A4 Layout — Print/PDF | Print dimensions might still be constrained by screen ratio; fix: remove `aspect-ratio` from base rule entirely and only use it inside `@media screen` |
| A2 | `config.show_seal !== false` correctly handles missing (undefined) flag as true-ish | Field Conditionals | If config omits `show_seal`, undefined !== false → still shows; that's correct behavior per schema defaults |

---

## Sources

### Primary (HIGH confidence)
- MDN Web Docs — `aspect-ratio` CSS property, browser compatibility, `@page` CSS rule — verified during research session [VERIFIED: MDN]
- `/.github/TRD-v2-Workshop-Certificate-App.md` — canonical HTML/CSS/JS reference for this project [VERIFIED: read in full]
- Current `assets/app.js`, `assets/styles.css`, `config/certificate.config.json`, `data/sample.json` — actual codebase state [VERIFIED: read in full]
- `.gsd/phases/01-scaffold-config-system/01-CONTEXT.md` — locked Phase 01 decisions [VERIFIED: read in full]

### Secondary (MEDIUM confidence)
- MDN HTMLElement `onerror` event — `onerror` must be set before `src` for cache-hit scenarios [CITED: MDN — best practice referenced, not experimentally verified in this session]

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — pure web fundamentals, nothing new introduced
- Architecture: HIGH — TRD provides full reference; conflicts with actual codebase are fully identified
- Pitfalls: HIGH — derived from direct codebase inspection, not guesswork
- A4 print behavior: MEDIUM — `@page` behavior across browsers/html2pdf is well-documented but has known quirks (Phase 03 will deal with pdf rendering depth)

**Research date:** 2026-04-11
**Valid until:** 2026-06-01 (stable web standards — no expiry risk)
