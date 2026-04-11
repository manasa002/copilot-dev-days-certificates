---
phase: 02-certificate-rendering
plan: "02"
type: execute
wave: 1
depends_on: []
files_modified:
  - assets/styles.css
autonomous: true
requirements:
  - CERT-03
  - CERT-04
  - CERT-07

must_haves:
  truths:
    - ".certificate has aspect-ratio: 297 / 210 for correct landscape A4 proportions on screen"
    - "@media print block includes @page { size: A4 landscape; margin: 0; } rule"
    - "Certificate border uses var(--border-width) double var(--border-color) — no hardcoded colors"
    - "Headings (.cert-name, .cert-heading-label) use var(--font-heading); body text uses var(--font-body)"
  artifacts:
    - path: "assets/styles.css"
      provides: "All certificate card CSS + @media print A4 block"
      contains: "aspect-ratio: 297"
  key_links:
    - from: "assets/styles.css .certificate"
      to: "Phase 01 applyConfigVars() CSS custom properties"
      via: "var(--border-color), var(--primary-color), var(--font-heading)"
      pattern: "var\\(--border-color\\)"
---

<objective>
Append all certificate visual styles to `assets/styles.css` — the card layout at A4 landscape proportions, all `.cert-*` component styles, and a `@media print` block for actual PDF/print output.

Purpose: The HTML structure from plan 02-01 needs CSS to render as a recognizable A4 certificate. CSS variables set by Phase 01's `applyConfigVars()` are used throughout — no hardcoded theme colors.

Output:
- `.certificate-wrapper` centering + `.certificate` A4 card styles
- All `.cert-header`, `.cert-body`, `.cert-footer`, `.cert-bottom-row` component styles
- `@media print` block with `@page { size: A4 landscape; }`
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/execute-plan.md
@$HOME/.claude/get-shit-done/templates/summary.md
</execution_context>

<context>
@assets/styles.css
@config/certificate.config.json
</context>

<tasks>

<task type="auto">
  <name>Task 1: Append certificate card CSS to assets/styles.css</name>
  <files>assets/styles.css</files>
  <action>
Append the following CSS block to the END of `assets/styles.css`. Do NOT remove or modify any existing rules.

**CRITICAL:** Use ONLY these CSS variable names (set by Phase 01 `applyConfigVars()`):
- `var(--primary-color)`, `var(--secondary-color)`, `var(--background-color)`
- `var(--text-color)`, `var(--muted-color)`, `var(--border-color)`, `var(--border-width)`
- `var(--font-heading)`, `var(--font-body)`

Do NOT use `--cert-primary`, `--cert-border`, `--cert-accent`, or any `--cert-*` names.

```css
/* ========================================
   CERTIFICATE CARD
   ======================================== */

/* Centers the certificate card on-screen */
.certificate-wrapper {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 20px;
  max-width: 1050px;
  margin: 0 auto;
  width: 100%;
}

/* A4 landscape proportions on screen (297mm × 210mm) */
.certificate {
  width: 100%;
  aspect-ratio: 297 / 210;
  background: var(--background-color);
  border: var(--border-width) double var(--border-color);
  font-family: var(--font-body);
  color: var(--text-color);
  display: flex;
  flex-direction: column;
  padding: 40px 60px;
  box-sizing: border-box;
  position: relative;
  box-shadow: 0 4px 24px rgba(0, 0, 0, 0.12);
}

/* Header: org logo above org name, centered */
.cert-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  margin-bottom: 16px;
}

.cert-logo {
  max-height: 60px;
  max-width: 200px;
  object-fit: contain;
}

.cert-org-name {
  font-family: var(--font-body);
  font-weight: 700;
  font-size: 0.8rem;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--primary-color);
  margin: 0;
}

/* Title block: ornament line — heading label — ornament line */
.cert-title-block {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 20px;
}

.cert-ornament-line {
  flex: 1;
  height: 1px;
  background: var(--secondary-color);
}

.cert-heading-label {
  font-family: var(--font-heading);
  font-size: 1rem;
  font-style: italic;
  color: var(--primary-color);
  white-space: nowrap;
  margin: 0;
}

/* Body: pre-name text, recipient name, post-name, workshop, description */
.cert-body {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  gap: 6px;
}

.cert-pre-name-text,
.cert-post-name-text {
  font-family: var(--font-body);
  font-size: 0.875rem;
  color: var(--text-color);
  margin: 0;
}

.cert-name {
  font-family: var(--font-heading);
  font-size: 2.4rem;
  font-weight: 700;
  color: var(--primary-color);
  margin: 4px 0;
  line-height: 1.2;
}

.cert-workshop {
  font-family: var(--font-heading);
  font-size: 1.2rem;
  font-weight: 400;
  font-style: italic;
  color: var(--primary-color);
  margin: 4px 0 0;
}

.cert-description {
  font-family: var(--font-body);
  font-size: 0.78rem;
  color: var(--text-color);
  max-width: 60%;
  text-align: center;
  font-style: italic;
  margin: 4px 0 0;
}

/* Footer: date left, seal center, signature right */
.cert-footer {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-top: 16px;
}

.cert-footer-left,
.cert-footer-right {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
  min-width: 130px;
}

.cert-footer-center {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.cert-date {
  font-family: var(--font-body);
  font-size: 0.875rem;
  font-weight: 600;
  color: var(--text-color);
}

.cert-seal {
  max-height: 80px;
  max-width: 80px;
  object-fit: contain;
}

.cert-signature {
  max-height: 60px;
  max-width: 160px;
  object-fit: contain;
}

.cert-authorized-name {
  font-family: var(--font-body);
  font-size: 0.875rem;
  font-weight: 700;
  color: var(--text-color);
  margin: 0;
}

.cert-footer-label {
  font-family: var(--font-body);
  font-size: 0.68rem;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: var(--muted-color);
}

/* Bottom row: certificate ID muted text */
.cert-bottom-row {
  text-align: center;
  margin-top: 8px;
}

.cert-id-label {
  font-family: var(--font-body);
  font-size: 0.62rem;
  color: var(--muted-color);
  letter-spacing: 0.05em;
  margin: 0;
}
```
  </action>
  <verify>
    <automated>(Select-String -Path 'assets/styles.css' -Pattern 'aspect-ratio: 297').Count -eq 1</automated>
  </verify>
  <done>
    - .certificate-wrapper, .certificate, and all .cert-* classes are present in styles.css
    - aspect-ratio: 297 / 210 is set on .certificate
    - Border uses var(--border-width) double var(--border-color)
    - No --cert-* variable names appear anywhere
  </done>
</task>

<task type="auto">
  <name>Task 2: Append @media print A4 landscape block</name>
  <files>assets/styles.css</files>
  <action>
Append the following `@media print` block to the END of `assets/styles.css`, after the certificate card CSS added in Task 1.

```css
/* ========================================
   PRINT / PDF (A4 Landscape)
   ======================================== */

@media print {
  @page {
    size: A4 landscape;
    margin: 0;
  }

  body {
    margin: 0;
    background: white;
  }

  /* Hide all views except certificate during print */
  .view {
    display: none !important;
  }

  #certificate-view {
    display: flex !important;
    justify-content: center;
    align-items: flex-start;
  }

  .certificate-wrapper {
    padding: 0;
    max-width: none;
  }

  /* Override screen aspect-ratio with exact physical dimensions */
  .certificate {
    width: 297mm;
    height: 210mm;
    aspect-ratio: auto;
    padding: 14mm 20mm;
    box-shadow: none;
  }
}
```
  </action>
  <verify>
    <automated>(Select-String -Path 'assets/styles.css' -Pattern '@page').Count -eq 1 AND (Select-String -Path 'assets/styles.css' -Pattern 'A4 landscape').Count -eq 1</automated>
  </verify>
  <done>
    - @media print block exists at end of styles.css
    - @page { size: A4 landscape; margin: 0; } is present
    - .certificate inside print has width: 297mm, height: 210mm, aspect-ratio: auto
  </done>
</task>

</tasks>

<threat_model>
## Trust Boundaries

| Boundary | Description |
|----------|-------------|
| CSS variables ← JS | `applyConfigVars()` writes CSS variable values from config; CSS consumes them |

## STRIDE Threat Register

| Threat ID | Category | Component | Disposition | Mitigation Plan |
|-----------|----------|-----------|-------------|-----------------|
| T-02-CSS-01 | Tampering | CSS custom properties | accept | Variables are set server-side from config; values are not user-controlled in this phase |
</threat_model>

<verification>
After both tasks:
- styles.css contains `.certificate { aspect-ratio: 297 / 210 }`
- styles.css contains `@media print` with `@page { size: A4 landscape }`
- Grep for `--cert-` in styles.css returns zero matches (no forbidden variable names)
- (Select-String -Path 'assets/styles.css' -Pattern '\-\-cert\-').Count -eq 0
</verification>

<success_criteria>
- .certificate has aspect-ratio: 297 / 210 on screen; 297mm × 210mm override in @media print
- All theme values consumed via CSS custom properties from Phase 01 (no hardcoded hex values)
- @media print block suppresses all non-certificate elements and sets exact A4 landscape page size
</success_criteria>

<output>
After completion, create `.gsd/phases/02-certificate-rendering/02-02-certificate-layout-SUMMARY.md`
</output>
