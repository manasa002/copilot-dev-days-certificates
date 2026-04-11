---
phase: 02-certificate-rendering
plan: "04"
type: execute
wave: 2
depends_on:
  - "02-01"
  - "02-03"
files_modified:
  - assets/app.js
autonomous: true
requirements:
  - CERT-01
  - CERT-02
  - CERT-05
  - CERT-06
  - DATA-04

must_haves:
  truths:
    - "/?id=sample renders a complete certificate with name, workshop, date, org name, logo, seal, signature, and muted certificate ID"
    - "/?id=nonexistent shows an error message that quotes the attempted id in #error-detail"
    - "Removing seal_url from config hides #cert-seal cleanly — no broken image icon"
    - "config.show_description: false hides #cert-description even when attendee.description exists"
    - "Certificate ID appears as muted small text via #cert-id-label, using id_label from config"
  artifacts:
    - path: "assets/app.js"
      provides: "renderCertificateView, setImageGraceful, showError(id)"
      contains: "renderCertificateView"
  key_links:
    - from: "init() after Promise.all"
      to: "renderCertificateView(config, attendee)"
      via: "direct call in id-present branch"
      pattern: "renderCertificateView"
    - from: "setImageGraceful()"
      to: "img.onerror handler"
      via: "el.onerror set BEFORE el.src assignment"
      pattern: "onerror"
    - from: "init() catch block"
      to: "showError(id)"
      via: "replace showView('error-view') with showError(id)"
      pattern: "showError"
---

<objective>
Add `setImageGraceful()`, `renderCertificateView(config, attendee)`, and `showError(id)` to `assets/app.js`. Also update the one line in `init()`'s catch block from `showView('error-view')` to `showError(id)`.

Purpose: This plan wires the data (from plan 02-03) to the HTML structure (from plan 02-01). After this plan, visiting `/?id=sample` renders a complete certificate, and visiting `/?id=nonexistent` shows a contextual error message.

Dependency reason:
- Requires 02-01: `renderCertificateView` accesses HTML IDs (`cert-name`, `cert-logo`, etc.) that 02-01 created
- Requires 02-03: `renderCertificateView` is called from `init()` which 02-03 restructured; `showError(id)` replaces the `showView` call 02-03 left in the catch block

Output:
- `setImageGraceful(id, src)` — graceful image load with onerror before src
- `renderCertificateView(config, attendee)` — populates all cert-* slots
- `showError(id)` — contextual error with quoted certificate ID
- `init()` catch updated: `showView('error-view')` → `showError(id)`
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/execute-plan.md
@$HOME/.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@assets/app.js
@index.html
@config/certificate.config.json
@data/sample.json
</context>

<interfaces>
<!-- Config keys (flat schema — use these exact paths, no config.certificate.* nesting) -->
<!-- From config/certificate.config.json -->

Config field → HTML element ID mapping:
  config.certificate_title  → #cert-heading-label  (textContent)
  config.org_name           → #cert-org-name        (textContent)
  config.org_name           → #cert-logo            (alt attribute)
  config.logo_url           → #cert-logo            (src via setImageGraceful)
  config.seal_url           → #cert-seal            (src via setImageGraceful, only if config.show_seal !== false)
  config.signature_url      → #cert-signature       (src via setImageGraceful)
  config.pre_name_text      → #cert-pre-name-text   (textContent)
  config.post_name_text     → #cert-post-name-text  (textContent)
  config.date_label         → #cert-date-label      (textContent)
  config.issued_by_label    → #cert-sig-label       (textContent)
  config.signature_name     → #cert-authorized-name (textContent)
  config.id_label           → #cert-id-label        (prefix, fallback 'Certificate ID')

Attendee field → HTML element ID mapping:
  attendee.name             → #cert-name            (textContent)
  attendee.workshop         → #cert-workshop        (textContent)
  attendee.date             → #cert-date            (textContent)
  attendee.date_iso         → #cert-date            (datetime attribute)
  attendee.description      → #cert-description     (textContent, visibility gated by config.show_description)
  attendee.certificate_id   → #cert-id-label        (appended after id_label prefix)

Conditional logic:
  show_description: config.show_description && attendee.description → show #cert-description (remove 'hidden')
  show_seal: config.show_seal === false OR !config.seal_url → call setImageGraceful with undefined/null → hides element
</interfaces>

<tasks>

<task type="auto">
  <name>Task 1: Add setImageGraceful() and renderCertificateView()</name>
  <files>assets/app.js</files>
  <action>
Add the following two functions to `assets/app.js`. Place them after the `fetchAttendee` / `validateAttendee` functions added by plan 02-03, before `init()`.

**setImageGraceful** — CRITICAL: set `onerror` BEFORE assigning `src`. This prevents the broken image icon from flashing if the image 404s.

```javascript
/**
 * Set an image src gracefully — hides the element if src is empty or the image 404s.
 * IMPORTANT: onerror must be assigned before src to catch immediate failures.
 */
function setImageGraceful(id, src) {
  var el = document.getElementById(id);
  if (!el) return;
  if (!src) {
    el.style.display = 'none';
    return;
  }
  el.onerror = function () {
    this.style.display = 'none';
  };
  el.src = src;
}
```

**renderCertificateView** — maps all config + attendee fields to their HTML slots using `document.getElementById`. Use the exact field mapping from the interfaces block above. No template literals required — plain string concatenation is fine.

```javascript
/**
 * Populate all certificate HTML slots from config and attendee data.
 * Requires all cert-* IDs to exist in index.html (created by plan 02-01).
 *
 * Config keys used (flat schema, no config.certificate.* nesting):
 *   certificate_title, org_name, logo_url, seal_url, signature_url,
 *   pre_name_text, post_name_text, date_label, issued_by_label,
 *   signature_name, id_label, show_description, show_seal
 */
function renderCertificateView(config, attendee) {
  // Org header
  var orgNameEl = document.getElementById('cert-org-name');
  if (orgNameEl) orgNameEl.textContent = config.org_name || '';

  setImageGraceful('cert-logo', config.logo_url);
  var logoEl = document.getElementById('cert-logo');
  if (logoEl && config.org_name) logoEl.alt = config.org_name;

  // Title block
  var headingEl = document.getElementById('cert-heading-label');
  if (headingEl) headingEl.textContent = config.certificate_title || 'Certificate';

  // Body: text labels
  var preNameEl = document.getElementById('cert-pre-name-text');
  if (preNameEl) preNameEl.textContent = config.pre_name_text || '';

  var nameEl = document.getElementById('cert-name');
  if (nameEl) nameEl.textContent = attendee.name;

  var postNameEl = document.getElementById('cert-post-name-text');
  if (postNameEl) postNameEl.textContent = config.post_name_text || '';

  var workshopEl = document.getElementById('cert-workshop');
  if (workshopEl) workshopEl.textContent = attendee.workshop;

  // Description: show only when config.show_description is truthy AND attendee has description
  var descEl = document.getElementById('cert-description');
  if (descEl) {
    if (config.show_description && attendee.description) {
      descEl.textContent = attendee.description;
      descEl.classList.remove('hidden');
    } else {
      descEl.classList.add('hidden');
    }
  }

  // Footer: date
  var dateEl = document.getElementById('cert-date');
  if (dateEl) {
    dateEl.textContent = attendee.date;
    dateEl.setAttribute('datetime', attendee.date_iso);
  }

  var dateLabelEl = document.getElementById('cert-date-label');
  if (dateLabelEl) dateLabelEl.textContent = config.date_label || '';

  // Footer: seal (hidden if show_seal === false or seal_url missing)
  if (config.show_seal === false) {
    var sealEl = document.getElementById('cert-seal');
    if (sealEl) sealEl.style.display = 'none';
  } else {
    setImageGraceful('cert-seal', config.seal_url);
  }

  // Footer: signature
  setImageGraceful('cert-signature', config.signature_url);

  var authorizedNameEl = document.getElementById('cert-authorized-name');
  if (authorizedNameEl) authorizedNameEl.textContent = config.signature_name || '';

  var sigLabelEl = document.getElementById('cert-sig-label');
  if (sigLabelEl) sigLabelEl.textContent = config.issued_by_label || '';

  // Bottom row: certificate ID
  var idLabelEl = document.getElementById('cert-id-label');
  if (idLabelEl) {
    var prefix = config.id_label || 'Certificate ID';
    idLabelEl.textContent = prefix + ': ' + attendee.certificate_id;
  }
}
```
  </action>
  <verify>
    <automated>(Select-String -Path 'assets/app.js' -Pattern 'renderCertificateView|setImageGraceful').Count -ge 4</automated>
  </verify>
  <done>
    - setImageGraceful(id, src) is defined, sets onerror before src, hides element for empty src
    - renderCertificateView(config, attendee) is defined and populates all 15 cert-* slots
    - show_description flag gates #cert-description visibility
    - show_seal flag gates #cert-seal visibility
    - (config.id_label || 'Certificate ID') + ': ' + attendee.certificate_id appears in cert-id-label
  </done>
</task>

<task type="auto">
  <name>Task 2: Add showError(id) and update init() catch block</name>
  <files>assets/app.js</files>
  <action>
Two changes to `assets/app.js`:

**Change 1 — Add showError(id) function.** Add this function after `renderCertificateView`:

```javascript
/**
 * Show the error view with an optional certificate-ID-specific message.
 * When id is falsy (e.g. config load failed), the hardcoded HTML message is preserved.
 * When id is truthy (certificate not found), updates #error-message and populates #error-detail.
 */
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

**Change 2 — Update init() catch block.** In the `init()` function (restructured by plan 02-03), find the catch block:

```javascript
  } catch (err) {
    console.error('[App] init error:', err);
    showView('error-view');   // <-- this line
  }
```

Replace the `showView('error-view')` call with `showError(id)`:

```javascript
  } catch (err) {
    console.error('[App] init error:', err);
    showError(id);
  }
```

The `if (id && msgEl)` guard in showError ensures:
- When visiting `/?id=nonexistent` → error message says "Certificate not found" + detail quotes the id
- When config itself fails to load (no id) → `showError(null)` is called, which skips the guard and preserves the hardcoded "Could not load configuration. Please try again." text from index.html
  </action>
  <verify>
    <automated>(Select-String -Path 'assets/app.js' -Pattern 'function showError').Count -eq 1 AND (Select-String -Path 'assets/app.js' -Pattern 'showError\(id\)').Count -ge 1</automated>
  </verify>
  <done>
    - showError(id) function is defined in app.js
    - showError(id) updates #error-message to 'Certificate not found.' when id is truthy
    - showError(id) populates and unhides #error-detail with the attempted id when id is truthy
    - showError(null/undefined) preserves the hardcoded error message text
    - init() catch block calls showError(id) instead of showView('error-view')
  </done>
</task>

</tasks>

<threat_model>
## Trust Boundaries

| Boundary | Description |
|----------|-------------|
| URL id param → DOM | Sanitized `id` is written into error message via textContent — not innerHTML |
| Config values → DOM | Config strings (org_name, etc.) written via textContent — not innerHTML |

## STRIDE Threat Register

| Threat ID | Category | Component | Disposition | Mitigation Plan |
|-----------|----------|-----------|-------------|-----------------|
| T-02-04-01 | XSS | renderCertificateView textContent | mitigate | ALL field assignments use `.textContent` (not `.innerHTML`). textContent is XSS-safe — sanitizes HTML entities automatically. |
| T-02-04-02 | XSS | showError id in error-detail | mitigate | `detailEl.textContent = '...' + id` uses textContent — malicious HTML in id param is rendered as plain text |
| T-02-04-03 | XSS | setImageGraceful src attribute | mitigate | `el.src = src` — browsers enforce same-origin for image src; URL values come from config (not user input). Phase 01 sanitizeId() already strips non-alphanumeric characters from the id param. |
</threat_model>

<verification>
After both tasks, in browser:
1. Visit /?id=sample (requires data/sample.json to exist) — certificate renders with all fields populated
2. Visit /?id=nonexistent — error view shows "Certificate not found." with detail quoting "nonexistent"
3. Visit / (no id) — search view shows (unchanged behavior)
4. Temporarily remove seal_url value from config — cert renders without broken image icon in seal position

Automated:
- (Select-String -Path 'assets/app.js' -Pattern 'renderCertificateView|setImageGraceful|showError').Count -ge 6
- (Select-String -Path 'assets/app.js' -Pattern '\.innerHTML').Count -eq 0  (all assignments use textContent)
</verification>

<success_criteria>
- renderCertificateView(config, attendee) populates all 15 cert-* slots with correct data
- setImageGraceful sets onerror → then src; hides element when src is empty
- showError(id) shows "Certificate not found" + detail with attempted id when id is truthy
- showError(null) preserves the default hardcoded message (config load failure case)
- init() catch calls showError(id)
- No innerHTML usage anywhere — all DOM writes use textContent or setAttribute
</success_criteria>

<output>
After completion, create `.gsd/phases/02-certificate-rendering/02-04-certificate-render-SUMMARY.md`
</output>
