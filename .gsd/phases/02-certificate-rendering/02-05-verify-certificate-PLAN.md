---
phase: 02-certificate-rendering
plan: "05"
type: execute
wave: 3
depends_on:
  - "02-04"
files_modified: []
autonomous: false
requirements:
  - APP-02
  - APP-05
  - DATA-03
  - DATA-04
  - CERT-01
  - CERT-02
  - CERT-03
  - CERT-04
  - CERT-05
  - CERT-06
  - CERT-07

must_haves:
  truths:
    - "Visiting /?id=sample renders a complete certificate with all fields visible"
    - "Visiting /?id=nonexistent shows 'Certificate not found.' with the attempted id quoted"
    - "Certificate proportions match A4 landscape in browser dev tools ruler"
    - "Playfair Display headings and Lato body text are visibly distinct"
    - "Certificate ID appears as muted small text at the bottom"
  artifacts:
    - path: "index.html"
      provides: "Certificate HTML"
      contains: "cert-name"
    - path: "assets/styles.css"
      provides: "A4 layout + print"
      contains: "aspect-ratio"
    - path: "assets/app.js"
      provides: "Full render pipeline"
      contains: "renderCertificateView"
  key_links: []
---

<objective>
Human verification that all four implementation plans (02-01 through 02-04) combine to deliver a visually correct, fully functional certificate rendering experience.

Purpose: Certificate rendering is a visual product — automated grep checks verify code presence but cannot confirm the certificate looks correct or that all fields are wired in the right places. This checkpoint verifies the end-to-end user experience before the phase is declared complete.

Output: Human confirmation that Phase 02 success criteria are met (or a list of issues to fix).
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/execute-plan.md
@$HOME/.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@index.html
@assets/app.js
@assets/styles.css
@config/certificate.config.json
@data/sample.json
</context>

<tasks>

<task type="checkpoint:human-verify" gate="blocking">
  <name>Task 1: Verify certificate rendering end-to-end</name>
  <what-built>
    Plans 02-01 through 02-04 collectively deliver:
    - Certificate HTML structure with 15 field slots (02-01)
    - A4 landscape CSS layout using Phase 01 CSS vars (02-02)
    - Parallel fetch via Promise.all, fetchAttendee, validateAttendee (02-03)
    - renderCertificateView, setImageGraceful, showError with ID quoting (02-04)
  </what-built>
  <how-to-verify>
Start a local server from the project root (e.g. `python -m http.server 8080` or Live Server in VS Code), then verify each scenario:

**Scenario 1 — Certificate renders correctly**
1. Open `http://localhost:8080/?id=sample`
2. Confirm you see a certificate card (not a blank page or loading spinner)
3. Confirm these fields are visible and correct:
   - Org name: "Acme Workshop Co."
   - Certificate title: "Certificate of Completion"
   - Recipient name in large Playfair Display font
   - Workshop name in italic below the name
   - Date of completion on the bottom-left
   - Seal image (bottom center) — check it loads
   - Signature image (bottom right)
   - Authorized by name: "Dr. Jane Smith"
   - Certificate ID in small muted text at the very bottom

**Scenario 2 — Error state with ID quoting**
1. Open `http://localhost:8080/?id=nobody-here`
2. Confirm the error view shows: "Certificate not found."
3. Confirm a second line reads: "We could not find a certificate for: nobody-here"
4. Confirm no unhandled JS exception in the browser console

**Scenario 3 — A4 proportions**
1. Open `http://localhost:8080/?id=sample`
2. Open browser dev tools (F12) → Elements → hover over `.certificate` element
3. Confirm the highlighted box has roughly 2:1.4 width:height ratio (297:210)
   OR: inspect the Computed styles and confirm `aspect-ratio: 297 / 210` is applied

**Scenario 4 — Description flag**
1. Temporarily set `show_description: false` in `config/certificate.config.json`
2. Hard-refresh `/?id=sample`
3. Confirm the description is NOT visible even though `data/sample.json` has a description value
4. Restore `show_description: true` before finishing

**Scenario 5 — Missing image graceful hide**
1. Temporarily set `seal_url: ""` (empty string) in config
2. Hard-refresh `/?id=sample`
3. Confirm NO broken image icon appears where the seal was — the space is simply empty
4. Restore the original seal_url
  </how-to-verify>
  <resume-signal>
Type "approved" if all 5 scenarios pass.
Type "issues" followed by which scenarios failed and what you observed if anything is wrong.
  </resume-signal>
</task>

</tasks>

<verification>
All 5 scenarios verified interactively by the human:
1. Certificate renders with all fields from config + data/sample.json
2. Invalid ID shows error message quoting the attempted ID
3. .certificate has A4 landscape proportions visible in dev tools
4. show_description: false hides the description field
5. Empty seal_url produces no broken image icon
</verification>

<success_criteria>
All Phase 02 ROADMAP success criteria are confirmed:
- [x] /?id=sample renders complete certificate with all fields
- [x] Certificate is visually landscape A4 proportions
- [x] /?id=nonexistent shows clear error message quoting the attempted ID
- [x] Missing seal produces no broken image icon
- [x] show_description: false hides the description field
</success_criteria>

<output>
After human approval, create `.gsd/phases/02-certificate-rendering/02-05-verify-certificate-SUMMARY.md`
</output>
