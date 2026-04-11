---
phase: 02-certificate-rendering
plan: "01"
type: execute
wave: 1
depends_on: []
files_modified:
  - index.html
autonomous: true
requirements:
  - CERT-01
  - CERT-06
  - DATA-04

must_haves:
  truths:
    - "#certificate-view contains .certificate-wrapper > .certificate with all 15 id='cert-*' HTML slots"
    - "#error-view <p class='error-message'> has id='error-message'; a new hidden <p id='error-detail'> sibling exists"
  artifacts:
    - path: "index.html"
      provides: "Certificate HTML structure with all field slots"
      contains: "id=\"cert-name\""
  key_links:
    - from: "index.html #cert-name, #cert-workshop, #cert-date, etc."
      to: "assets/app.js renderCertificateView() (plan 02-04)"
      via: "document.getElementById() in renderCertificateView"
      pattern: "id=\"cert-"
    - from: "index.html #error-message, #error-detail"
      to: "assets/app.js showError(id) (plan 02-04)"
      via: "document.getElementById('error-message').textContent"
      pattern: "id=\"error-"
---

<objective>
Add the static HTML structure for the certificate card inside `#certificate-view`, and update `#error-view` with element IDs that the JavaScript layer will use to display dynamic error details.

Purpose: Plan 02-04's `renderCertificateView()` depends on these exact HTML IDs existing before it populates them. This plan creates the slots — no JavaScript is touched here.

Output:
- `#certificate-view` populated with `.certificate-wrapper > .certificate` containing all field slots
- `#error-view` updated with `id="error-message"` and `id="error-detail"`
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/execute-plan.md
@$HOME/.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@index.html
@config/certificate.config.json
@data/sample.json
</context>

<tasks>

<task type="auto">
  <name>Task 1: Insert certificate HTML structure into #certificate-view</name>
  <files>index.html</files>
  <action>
In `index.html`, replace the entire `<div id="certificate-view" class="view hidden"></div>` block (currently empty with a comment) with the following. Keep the outer div's exact attributes (`id="certificate-view"`, `class="view hidden"`) unchanged — the `hidden` class is removed by JS at runtime.

Insert this content INSIDE the `#certificate-view` div:

```html
<div id="certificate-view" class="view hidden">
  <div class="certificate-wrapper">
    <div class="certificate" id="certificate">

      <div class="cert-header">
        <img id="cert-logo" src="" alt="" class="cert-logo" />
        <p class="cert-org-name" id="cert-org-name"></p>
      </div>

      <div class="cert-title-block">
        <div class="cert-ornament-line"></div>
        <p class="cert-heading-label" id="cert-heading-label"></p>
        <div class="cert-ornament-line"></div>
      </div>

      <div class="cert-body">
        <p class="cert-pre-name-text" id="cert-pre-name-text"></p>
        <h1 class="cert-name" id="cert-name"></h1>
        <p class="cert-post-name-text" id="cert-post-name-text"></p>
        <h2 class="cert-workshop" id="cert-workshop"></h2>
        <p class="cert-description hidden" id="cert-description"></p>
      </div>

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

      <div class="cert-bottom-row">
        <p class="cert-id-label" id="cert-id-label"></p>
      </div>

    </div>
  </div>
</div>
```

The IDs that MUST be present are:
`cert-logo`, `cert-org-name`, `cert-heading-label`, `cert-pre-name-text`, `cert-name`, `cert-post-name-text`, `cert-workshop`, `cert-description`, `cert-date`, `cert-date-label`, `cert-seal`, `cert-signature`, `cert-authorized-name`, `cert-sig-label`, `cert-id-label`
  </action>
  <verify>
    <automated>(Select-String -Path 'index.html' -Pattern 'id="cert-').Count — must return 15</automated>
  </verify>
  <done>All 15 cert-* IDs are present in index.html inside #certificate-view</done>
</task>

<task type="auto">
  <name>Task 2: Update #error-view for dynamic error display</name>
  <files>index.html</files>
  <action>
In `index.html`, update the `#error-view` block. The current HTML is:

```html
<div id="error-view" class="view hidden" role="alert">
  <div class="error-container">
    <span class="error-icon" aria-hidden="true">⚠</span>
    <p class="error-message">Could not load configuration. Please try again.</p>
  </div>
</div>
```

Replace it with:

```html
<div id="error-view" class="view hidden" role="alert">
  <div class="error-container">
    <span class="error-icon" aria-hidden="true">⚠</span>
    <p class="error-message" id="error-message">Could not load configuration. Please try again.</p>
    <p class="error-detail hidden" id="error-detail"></p>
  </div>
</div>
```

Changes:
1. Add `id="error-message"` to the existing `<p class="error-message">` — do NOT change its default text
2. Add a new `<p class="error-detail hidden" id="error-detail"></p>` after it — empty and hidden by default
  </action>
  <verify>
    <automated>(Select-String -Path 'index.html' -Pattern 'id="error-message"').Count -eq 1 AND (Select-String -Path 'index.html' -Pattern 'id="error-detail"').Count -eq 1</automated>
  </verify>
  <done>
    - id="error-message" is on the existing <p> that retains its default text
    - id="error-detail" exists as a new hidden sibling <p> with no default text
  </done>
</task>

</tasks>

<verification>
After both tasks:
- Open index.html and confirm #certificate-view is no longer an empty div
- Confirm #error-view has both id="error-message" and id="error-detail"
- (Select-String -Path 'index.html' -Pattern 'id="cert-').Count returns 15
</verification>

<success_criteria>
- index.html #certificate-view contains full certificate HTML skeleton with 15 cert-* IDs
- index.html #error-view has id="error-message" (existing <p>) and id="error-detail" (new hidden <p>)
- No JavaScript was changed — this plan is HTML-only
</success_criteria>

<output>
After completion, create `.gsd/phases/02-certificate-rendering/02-01-certificate-structure-SUMMARY.md`
</output>
