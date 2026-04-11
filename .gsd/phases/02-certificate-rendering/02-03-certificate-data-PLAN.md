---
phase: 02-certificate-rendering
plan: "03"
type: execute
wave: 1
depends_on: []
files_modified:
  - assets/app.js
autonomous: true
requirements:
  - APP-02
  - APP-05
  - DATA-03
  - DATA-04

must_haves:
  truths:
    - "fetchAttendee(id) fetches data/{id}.json and throws a descriptive Error on non-200 response"
    - "validateAttendee() throws if any of the 6 required fields is missing: certificate_id, name, email, workshop, date, date_iso"
    - "init() uses Promise.all([fetchConfig(), fetchAttendee(id)]) when a certificate id is present in the URL"
    - "init() routes to search-view when no id param is present"
    - "init() catch block calls showView('error-view') — to be updated to showError(id) by plan 02-04"
  artifacts:
    - path: "assets/app.js"
      provides: "fetchAttendee, validateAttendee, restructured init()"
      contains: "Promise.all"
  key_links:
    - from: "init() id branch"
      to: "Promise.all([fetchConfig(), fetchAttendee(id)])"
      via: "parallel fetch before render"
      pattern: "Promise\\.all"
    - from: "fetchAttendee(id)"
      to: "data/{id}.json"
      via: "fetch API"
      pattern: "data/\\$\\{id\\}\\.json"
---

<objective>
Add `fetchAttendee(id)` and `validateAttendee(attendee)` functions to `assets/app.js`, then restructure `init()` to use `Promise.all()` for the certificate-id branch.

Purpose: The app currently has a placeholder in `init()` that calls `showView('certificate-view')` when an id is present. This plan replaces that placeholder with real parallel data fetching and routing — config + attendee JSON load simultaneously.

Output:
- `fetchAttendee(id)` — fetches `data/{id}.json`, throws on failure
- `validateAttendee(attendee)` — validates required fields
- Restructured `init()` — Promise.all parallel fetch, routes to search or certificate view
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/execute-plan.md
@$HOME/.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@assets/app.js
@config/certificate.config.json
@data/sample.json
</context>

<tasks>

<task type="auto">
  <name>Task 1: Add fetchAttendee() and validateAttendee() to app.js</name>
  <files>assets/app.js</files>
  <action>
Add the following two functions to `assets/app.js`. Place them near the existing `fetchConfig()` and `validateConfig()` functions for logical grouping.

```javascript
/**
 * Fetch attendee JSON from data/{id}.json.
 * Throws an Error if the file is not found or response is not ok.
 */
async function fetchAttendee(id) {
  var response = await fetch('data/' + id + '.json');
  if (!response.ok) {
    throw new Error('Certificate not found for: ' + id);
  }
  return response.json();
}

/**
 * Validate that all required attendee fields are present.
 * Throws a descriptive Error if any required field is missing.
 */
function validateAttendee(attendee) {
  var required = ['certificate_id', 'name', 'email', 'workshop', 'date', 'date_iso'];
  for (var i = 0; i < required.length; i++) {
    if (!attendee[required[i]]) {
      throw new Error('Attendee data is missing required field: ' + required[i]);
    }
  }
}
```

Notes:
- Uses `var` and function declarations consistent with the ES6+ but non-class style already in app.js
- `fetchAttendee` uses template-equivalent string concatenation (`'data/' + id + '.json'`) — or template literals if the existing codebase already uses them
- Throws on non-ok response so the `init()` catch block handles 404s
  </action>
  <verify>
    <automated>(Select-String -Path 'assets/app.js' -Pattern 'fetchAttendee|validateAttendee').Count -ge 4</automated>
  </verify>
  <done>
    - fetchAttendee(id) is defined, fetches data/{id}.json, throws on non-200
    - validateAttendee(attendee) is defined, checks all 6 required fields
  </done>
</task>

<task type="auto">
  <name>Task 2: Restructure init() to use Promise.all for certificate branch</name>
  <files>assets/app.js</files>
  <action>
Replace the current `init()` function in `assets/app.js` with the restructured version below. The existing init() fetches config and then does a `showView('certificate-view')` placeholder when an id is present — replace the entire function body.

```javascript
async function init() {
  showView('loading-view');
  var rawId = getQueryParam('id');
  var id = rawId ? sanitizeId(rawId) : null;
  try {
    if (id) {
      // Certificate branch: load config + attendee data in parallel
      var results = await Promise.all([fetchConfig(), fetchAttendee(id)]);
      var config = results[0];
      var attendee = results[1];
      validateConfig(config);
      validateAttendee(attendee);
      applyConfigVars(config);
      if (config.site_title) {
        document.title = config.site_title;
      }
      renderCertificateView(config, attendee);
      showView('certificate-view');
    } else {
      // No id: show search view
      var config = await fetchConfig();
      validateConfig(config);
      applyConfigVars(config);
      if (config.site_title) {
        document.title = config.site_title;
      }
      showView('search-view');
    }
  } catch (err) {
    console.error('[App] init error:', err);
    showView('error-view');
  }
}
```

Notes:
- `renderCertificateView(config, attendee)` is called here but will be defined by plan 02-04 — this is a forward reference that will resolve when 02-04 runs. Do NOT define a stub — leave the call as written.
- The catch block calls `showView('error-view')` for now. Plan 02-04 will update this one line to `showError(id)` after defining the showError function.
- Uses `var results` instead of destructuring for ES5 compat consistency — or use `const [config, attendee]` if existing code already uses destructuring.
  </action>
  <verify>
    <automated>(Select-String -Path 'assets/app.js' -Pattern 'Promise\.all').Count -ge 1</automated>
  </verify>
  <done>
    - init() uses Promise.all([fetchConfig(), fetchAttendee(id)]) in the id-present branch
    - init() routes to showView('search-view') when id is null
    - init() catch block calls showView('error-view') (to be updated in plan 02-04)
    - renderCertificateView(config, attendee) is called in the id branch
  </done>
</task>

</tasks>

<threat_model>
## Trust Boundaries

| Boundary | Description |
|----------|-------------|
| URL → app | `id` param from `getQueryParam()` — already sanitized by existing `sanitizeId()` from Phase 01 |
| data/{id}.json → JS | Attendee JSON loaded from same-origin GitHub Pages static file |

## STRIDE Threat Register

| Threat ID | Category | Component | Disposition | Mitigation Plan |
|-----------|----------|-----------|-------------|-----------------|
| T-02-03-01 | Spoofing | fetchAttendee URL path | mitigate | `sanitizeId()` from Phase 01 strips special characters before use in path; only alphanumeric + `-` + `at` pattern allowed |
| T-02-03-02 | Information Disclosure | Non-200 error message | accept | Error message reveals "Certificate not found" — minimal disclosure, same-origin static files only |
| T-02-03-03 | Tampering | Attendee JSON fields | accept | Static files on GitHub Pages; no user-controlled write path exists |
</threat_model>

<verification>
After both tasks:
- (Select-String -Path 'assets/app.js' -Pattern 'Promise\.all').Count -ge 1
- (Select-String -Path 'assets/app.js' -Pattern 'fetchAttendee').Count -ge 2 (definition + call)
- (Select-String -Path 'assets/app.js' -Pattern 'validateAttendee').Count -ge 2 (definition + call)
- init() no longer contains its old placeholder showView('certificate-view') without the Promise.all
</verification>

<success_criteria>
- fetchAttendee(id) fetches data/{id}.json and throws an Error on non-200
- validateAttendee(attendee) validates all 6 required fields
- init() uses Promise.all for the id-present branch
- init() routes to search-view when no id param is present
- init() catch block calls showView('error-view') (placeholder — updated in 02-04)
</success_criteria>

<output>
After completion, create `.gsd/phases/02-certificate-rendering/02-03-certificate-data-SUMMARY.md`
</output>
