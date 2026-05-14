# QA Checklist — Landing Page Verification

Loaded during Step 7 (Phase 2) of the shopify-landing skill.

---

## Pre-QA setup

Navigate to the live product URL using Claude in Chrome:
```
https://STORE_URL/products/PRODUCT_HANDLE
```

Take a full-page screenshot before starting checks.
Also open the competitor reference URL and take a screenshot for comparison.

---

## Section visibility checks

For each section in `SECTION_STRUCTURE`, verify it renders:

| Check | Method | Pass condition |
|-------|--------|----------------|
| Section is present | Scroll through full page | All sections visible, none missing |
| Section order | Compare to SECTION_STRUCTURE list | Matches exactly |
| No blank sections | Visual inspection | No empty containers or invisible blocks |
| No JS errors | Chrome DevTools console | Zero red errors on page load |

---

## Functional checks

| Element | Test | Pass condition |
|---------|------|----------------|
| Add to Cart button | Click it | Opens cart or redirects to checkout |
| Price display | Visual | Shows PRODUCT_PRICE correctly formatted |
| Variant picker | If applicable | Selects correctly, price updates |
| Page load speed | Subjective | Loads in under 3 seconds |

---

## Visual fidelity checks

Compare against competitor screenshot:

| Element | Check |
|---------|-------|
| Primary color | CTAs and accents match COLOR_PRIMARY |
| Background | Section fills match COLOR_SECONDARY |
| Headings | Font matches FONT_HEADING (serif/sans) |
| Body text | Font matches FONT_BODY |
| Section spacing | Padding feels similar to reference |
| Overall density | Layout density matches (minimal vs packed) |

---

## Social proof section check (if present)

| Check | Pass condition |
|-------|----------------|
| Reviews are visible | At least 3 reviews show |
| No third-party widget | No Judgeme, Okendo, Yotpo iframes present |
| Reviewer names and locations | Present on each review |
| Star ratings | Rendered in HTML/CSS, not images from an app |
| Review text | Custom written, not generic |

---

## Image checks

| Check | Pass condition |
|-------|----------------|
| No grey placeholder boxes | Zero empty image containers |
| No broken image icons | All `<img>` tags resolve |
| Brand color fallbacks | Any intentional empty slots use brand color background |

---

## Fix protocol

For each failed check:
1. Identify which `.liquid` file controls that element.
2. Patch the specific file via REST API (do not re-upload all files).
3. Hard-refresh the page in Chrome.
4. Re-check only the fixed element.
5. Log the fix in the handoff report.

If a fix requires more than 2 attempts on the same element → flag it in the handoff report as a known issue with reproduction steps.
