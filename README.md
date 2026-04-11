# Workshop Certificate App

A fully static, zero-backend web application that generates personalized, printable, and downloadable workshop completion certificates — hosted on GitHub Pages for free.

Every attendee gets a unique URL with a QR code. Scanning it opens their certificate instantly.

---

## How to Claim Your Certificate

These are the steps to get your certificate published:

### Step 1 — Fork this repository

Click **Fork** (top-right of this page) to create a copy under your GitHub account.

### Step 2 — Create your data file

In your fork, navigate to the `data/` folder and create a new file named using your email address:

**Naming convention:** convert your email to a safe ID:
- `jane.doe@gmail.com` → `jane-doe-at-gmail-com.json`
- Replace `@` with `-at-`
- Replace `.` with `-`
- Replace `+` with `-plus-`
- Lowercase everything

**File contents** (`data/your-name-at-email-com.json`):

```json
{
  "certificate_id": "your-name-at-email-com",
  "name": "Your Full Name",
  "email": "your@email.com",
  "workshop": "Github Copilot Dev Days 2026 - Hyderabad",
  "date": "April 5, 2026",
  "date_iso": "2026-04-05",
  "description": "Completed 16 hours of hands-on training covering supervised learning, neural networks, and model evaluation."
}
```

> `certificate_id` must exactly match your filename (without `.json`).

### Step 3 — Open a Pull Request

Go back to the **original** repository (not your fork) and open a Pull Request from your fork's `main` branch.

Fill in the PR template checklist and submit. The organizer will review and merge it.

### Step 4 — Get your certificate URL

After your PR is merged, your certificate will be live at:

```
https://<org>.github.io/certificates/?id=your-name-at-email-com
```

You can also search by email at the homepage.

---

## Run Locally

No build step required. Just serve the folder:

```bash
# Using Node.js
npx serve .

# Using Python
python -m http.server 8080
```

Then open `http://localhost:8080/?id=sample` to preview the sample certificate.

---

## Customizing for Your Organization

Edit only `config/certificate.config.json` — no HTML changes needed:

| Field | Description |
|---|---|
| `org_name` | Organization name shown on the certificate |
| `logo_url` | Path to logo image (PNG with transparency, ~200×60px) |
| `seal_url` | Path to seal image (circular PNG, ~100×100px) |
| `signature_url` | Path to signature image (~160×60px, PNG) |
| `primary_color` | Main brand color (hex) |
| `secondary_color` | Accent/gold color (hex) |
| `certificate_title` | e.g. "Certificate of Participation" |
| `signature_name` | Name printed under the signature |
| `org_website` | Encoded in the QR code for verification |
| `seal_label` | Text below the seal, e.g. "Certified" |
| `twitter_handle` | For Twitter Card meta tags |
| `show_qr` | `true` / `false` — toggle QR code |
| `show_description` | `true` / `false` — toggle description paragraph |

Replace the images in `assets/images/` with your own and update the paths in the config.

---

## Forking for a New Workshop

1. Fork this repo
2. Edit `config/certificate.config.json` with your org details
3. Replace `assets/images/logo.png`, `seal.png`, `signature.png`
4. Enable GitHub Pages on your fork (Settings → Pages → `main` branch, `/ (root)`)
5. Drop attendee JSON files in `data/` as PRs are merged

Zero HTML changes required.

---

*Built with HTML5 + CSS3 + Vanilla JS. Hosted on GitHub Pages. $0/month.*
