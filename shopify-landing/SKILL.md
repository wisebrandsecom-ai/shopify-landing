# Shopify Landing Page Skill

## ⛔ HARDCODED RULES — These override everything else. Always.

1. **NO EMOJIS** — Never use emoji as icons. Always use inline SVG. Only exception: competitor explicitly uses emojis in that exact element.

0. **NO PLACEHOLDERS EVER** — Never leave `{# ... #}`, `<!-- placeholder -->`, `[IMAGE HERE]`, `{# LEFT GALLERY #}`, `{# RIGHT BUY BOX #}`, or ANY unfilled comment in any .liquid file. If a section is not complete, do not upload it. Complete it first.

2. **BUY BOX = custom-product section** — Build `sections/custom-product.liquid` from scratch. It contains gallery (55% width) + buy box (45% width) in ONE section. The `order` array in the template JSON contains ONLY `custom-product`. Remove `main-product` entirely. Never split buy box elements into separate page sections.

3. **FOOTER = custom-footer section** — Build `sections/custom-footer.liquid` from scratch. Comment out `{% section 'footer' %}` in `layout/theme.liquid`. Replace with `{% section 'custom-footer' %}`. Never use CSS overrides on native footer.

4. **ANNOUNCEMENT BAR = custom section** — Most themes use plain-text announcement bars. Always create a custom section with full HTML support. Disable native bar. Add custom bar to `layout/theme.liquid`.

5. **IMAGES ARE MANDATORY** — Run Higgsfield Phase B after Step 6. Every image slot in every section must have a real image before delivery. No grey boxes.

6. **ALL CSS IS MOBILE-FIRST** — Base styles = mobile (no media query). Desktop = `@media (min-width: 750px)`. Font min 14px body, 22px headings on mobile.

7. **VERIFY BEFORE PROCEEDING** — Re-read every file after writing it. Confirm the save before moving to the next step.

8. **BRAND NAME REPLACEMENT (LANDING COPY ONLY)** — Replace competitor brand name with `BRAND_NAME` in: section headings, copy, alt text, schema settings, footer, announcement bar. NOT in packaging artwork when `VISUAL_MODE = REPLICATE` — keep original packaging label exactly. See higgsfield-photos.md VISUAL_MODE rule.

13. **FAIL LOUD, NOT SILENT** — If any output diverges from the reference (wrong layout, wrong package, wrong language, garbled text, missing images), STOP and print a DIVERGENCE REPORT. Wait for user input. Never paper over broken output.

15. **ONE-TO-ONE FIDELITY (Phase A)** — Phase A generates EXACTLY one image per file in PRODUCT_PHOTOS_FOLDER tagged `clean_packshot` or `lifestyle_with_pouch`. If folder has 9 valid files → 9 generate_image calls → 9 photos uploaded. No inventing scenes. No combining references. Before calling generate_image, print:
    ```
    generations_planned = N
    tagged_rows_in_A2   = N
    ```
    If they don't match → STOP, report DIVERGENCE.

16. **PRINT-BEFORE-ACT** — Before any generate_image call (Phase A or B), print the Pre-flight Checklist from `higgsfield-photos.md` with each box explicitly marked [x] or [ ]. Printing "preconditions ok" without the boxes = Rule 16 violation, run stops. — If any output diverges from the reference (wrong layout, wrong package, wrong language, garbled text, missing images), STOP the run and print a DIVERGENCE REPORT:
    ```
    DIVERGENCE: [what failed]
    Expected: [what reference shows]
    Generated/Built: [what came out]
    Likely cause: [specific reason]
    ```
    Wait for user input. Never paper over broken output by continuing silently.

14. **TEMPLATE OVERWRITE, NEVER MERGE** — Before uploading `templates/product.TEMPLATE_NAME.json`, delete any existing file with that name in the working theme first. Use a consistent naming convention for section IDs across builds (e.g. always `s01_`, `s02_` prefix). After upload, if storefront shows old section IDs, open the theme editor and save any section — this forces a CDN cache rebuild instantly. — Before uploading `templates/product.TEMPLATE_NAME.json`, check if a file with that name already exists in the working theme. If it does, delete it first or use a fresh TEMPLATE_NAME (e.g. `garlic-v2`). After upload, verify the section IDs in the storefront HTML match your template's order array. A mismatch means the wrong theme is being served — not cache.

9. **BUY BOX — NO {# #} COMMENTS, NO STICKY GALLERY** — In `custom-product.liquid`:
   - Never use `{# ... #}` comment syntax — Shopify Liquid does not support it. Use `{% comment %}...{% endcomment %}` or remove comments entirely.
   - The `.cp-gallery` must NOT have `position: sticky` — this causes the gallery to follow the user while scrolling. Remove it.
   - Layout: gallery on the LEFT (order: 1, 55%), buy box on the RIGHT (order: 2, 45%) using flexbox order. Both are static. This matches the standard competitor layout (image left, content right).
   - Mobile: `flex-direction: column`, gallery stacks ABOVE buy box (order resets to DOM order on mobile).

10. **NEVER INVENT SECTIONS** — Every section must exist in the competitor's page. If a section in the competitor uses a horizontal photo carousel → build a horizontal photo carousel. If it uses a timeline with cards → build that exact timeline. Never substitute with a different section type. Never add sections the competitor doesn't have.

11. **DETECT MEDIA TYPE EXACTLY** — Before building any section with media, identify from the competitor: is it a photo? a video? a GIF? a static illustration? Build exactly that type. If competitor shows photos in a slider → build a photo slider, not video thumbnails. "Video testimonials" are only used if the competitor explicitly shows video play buttons.

12. **HIGGSFIELD IS MANDATORY FOR PRODUCT PHOTOS** — Never upload reference images from PRODUCT_PHOTOS_FOLDER directly to Shopify. Reference images are composition guides only. Every product photo that goes to Shopify must be generated fresh in Higgsfield using PRODUCT_MEDIA_ID as the product reference.

---

## Flow overview

```
Step 0  — Auth (shopify store auth)
Step 0.5 — Gather config + analyze competitor PDF
Step 1  — Analyze competitor (read references/competitor-analysis.md)
Step 2  — Choose theme mode (A: duplicate / B: existing / C: live)
Step 3  — Create product template slug
Step 4  — Interview user (skip if all data known)
Step 5  — Create product in Shopify
Step 6  — Build sections (read ALL references before starting)
         6a: custom-product.liquid (buy box — RULE #2)
         6b: Below-fold sections (one .liquid file per section)
         6c: Higgsfield Phase B — fill ALL image slots (RULE #5)
Step 6.5 — Global theme: announcement bar + header + footer (RULES #3 #4)
Step 7  — QA
Step 8  — Higgsfield Phase A — product gallery photos
OUTPUT  — Live landing
```

---

## Step 0 — Auth

```bash
shopify store auth --store STORE_URL --scopes write_products,write_inventory,write_themes,write_content,write_files,write_metaobjects,write_metaobject_definitions,write_publications,write_translations,read_locations,read_analytics
```

Browser opens → user approves → confirm and continue.

Store session variables: `STORE_URL`, `OAUTH_CLIENT`, `OAUTH_SECRET`

---

## Step 0.5 — Global config

**Read ALL references before writing any code:**
- `references/competitor-analysis.md`
- `references/above-the-fold.md`
- `references/elixir-design-language.md`
- `references/section-schema-standards.md`
- `references/landing-designer.md`
- `references/higgsfield-photos.md`

**Session variables to set:**
```
OUTPUT_LANGUAGE    TEMPLATE_NAME      COLOR_PRIMARY
COLOR_TEXT         BRAND_TONE         PRODUCT_TITLE
PRODUCT_PRICE      COMPARE_AT_PRICE   COPY_STRATEGY
VISUAL_MODE        THEME_MODE         WORKING_THEME_ID
ICON_STYLE         (SVG or EMOJI — detect from competitor PDF)
PRODUCT_PHOTO      (file attached by user — used for Higgsfield Phase A)
BRAND_NAME              (your brand name — replaces competitor brand everywhere)
PRODUCT_DOC             (optional PDF with your product info — required for COPY_STRATEGY D)
                        Include: benefits, avatar, pain points, differentiators, FAQs, guarantee
PRODUCT_PHOTOS_FOLDER   (local desktop folder path — for product gallery photos)
                        e.g. /Users/mario/Desktop/fotos-producto
```

**Images — two separate sources, no Drive needed:**
- **Product gallery photos (Phase A):** `PRODUCT_PHOTOS_FOLDER` local folder + attached product PNG
- **Landing section images (Phase B):** extracted from competitor PDF + Chrome screenshots — no folder needed

**If PRODUCT_PHOTO is attached:** Store it immediately. It will be used in Step 8 as the reference image for all product photo generation. Every Higgsfield product shot must show this exact packaging.

**ICON_STYLE detection:** Look at competitor's trust badges, bullets, and pills. If they render as icons (not emoji characters) → `ICON_STYLE: SVG`. If they are emoji → `ICON_STYLE: EMOJI`. Default is SVG.

---

## Step 1 — Competitor analysis ⚠️ PRINT ANALYSIS BEFORE BUILDING ANYTHING

Read `references/competitor-analysis.md`. Use the PDF only — do not fetch the URL.

**Single source: PDF only.**
The PDF is the ground truth. Do not fetch the URL, do not open Chrome, do not inspect the live page.
COMPETITOR_URL is stored for reference only (e.g. for the user to verify) but is NOT fetched or visited.

Read the PDF thoroughly:
1. Extract section structure top to bottom
2. Extract all copy, colors, fonts visible
3. Extract buy box elements in order
4. Extract media types per section (photo/video/illustration)
5. Extract announcement bar, header, footer details

**If COPY_STRATEGY = D (different product competitor):**
Also read PRODUCT_DOC before extracting anything. Store as `product_research`:
- Avatar pain points (in their exact words if possible)
- Avatar desire / transformation
- Product benefits (top 5-7)
- Differentiators vs alternatives
- Tone of voice
- FAQs / objections
- Guarantee details

When building copy in Step 6b, use `product_research` as the ONLY copy source.
The competitor PDF provides: section order, visual structure, argument flow, section types.
The competitor PDF does NOT provide: claims, benefits, product details — those come from PRODUCT_DOC.
Every claim in the landing must be true for YOUR product, not the competitor's.

**MANDATORY: Print the full analysis before writing any code.**
Do not start Step 2 until this block is printed in full:

```
════════════════════════════════════════
  COMPETITOR ANALYSIS COMPLETE
════════════════════════════════════════
COLORS:
  Primary:     [exact hex from competitor]
  Secondary:   [exact hex]
  Background:  [exact hex]
  Text:        [exact hex]
  Accent:      [exact hex — usually for CTA buttons]

FONTS:
  Heading:     [Google Font name or system font]
  Body:        [Google Font name or system font]

ANNOUNCEMENT BAR:
  Type:        [static / scrolling_ticker / countdown / dual_bar]
  Bar 1 text:  [exact content]
  Bar 1 bg:    [hex]
  Bar 2 text:  [if exists]
  Bar 2 type:  [countdown / ticker / none]

HEADER:
  Logo:        [text or image]
  Sticky:      [yes/no]
  Background:  [hex]

BUY BOX ELEMENTS (in order):
  1. [element type — e.g. rating, title, bullets, bundle picker, ATC, trust badges, FAQ]
  2. ...

SECTIONS (numbered, top to bottom):
  1. [section type] — [brief description]
  2. [section type] — [brief description]
  ...

FOOTER:
  Background:  [hex]
  Columns:     [list]
  Has newsletter: [yes/no]
════════════════════════════════════════
```

**COLOR RULE: Use ONLY the competitor's exact colors above.**
Never use COLOR_PRIMARY from prompt unless VISUAL_MODE = CUSTOM.
If VISUAL_MODE = REPLICATE, the competitor's colors override everything.

---

## Step 2 — Theme mode

**Mode A — Duplicate base theme** (for A/B testing, batch)
```bash
curl -X POST "https://STORE_URL/admin/api/2024-01/themes.json" \
  -H "X-Shopify-Access-Token: ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"theme": {"name": "PRODUCT_TITLE", "role": "unpublished"}}'
```

**Mode B — Use existing theme** (same branding, new product)

**Mode C — Live theme** (publish directly — warns user)

Store theme ID as `WORKING_THEME_ID`.

---

## Step 3 — Template

```bash
# Create product template JSON
curl -X PUT "https://STORE_URL/admin/api/2024-01/themes/WORKING_THEME_ID/assets.json" \
  -H "X-Shopify-Access-Token: ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"asset": {"key": "templates/product.TEMPLATE_NAME.json", "value": "{\"sections\":{},\"order\":[]}"}}'
```

---

## Step 4 — Product interview

Skip entirely if: `PRODUCT_TITLE`, `PRODUCT_PRICE`, `COPY_STRATEGY`, `VISUAL_MODE` all known.

Otherwise ask once, grouped.

---

## Step 5 — Create product

Read `references/product-creator.md`.

**Out of stock prevention — mandatory:**
When creating the product via API, always set inventory:
```bash
# After creating product, get inventory_item_id from variant
# Then set inventory quantity
curl -X POST "https://STORE_URL/admin/api/2024-01/inventory_levels/set.json"   -H "X-Shopify-Access-Token: ACCESS_TOKEN"   -H "Content-Type: application/json"   -d '{"location_id": LOCATION_ID, "inventory_item_id": INVENTORY_ITEM_ID, "available": 999}'
```

Get location_id first:
```bash
curl "https://STORE_URL/admin/api/2024-01/locations.json"   -H "X-Shopify-Access-Token: ACCESS_TOKEN"
```

Also ensure `inventory_policy` is set to `"continue"` (sell when out of stock) OR set `"tracked": true` with quantity > 0.
Never leave inventory unset — Shopify defaults to 0 which shows as sold out.

---

## Step 6 — Build landing

### 6a — custom-product.liquid (buy box) ⚠️ RULE #2

Read `references/above-the-fold.md` — follow it exactly.

**Critical:**
- `sections/custom-product.liquid` — full section with gallery + buy box
- `assets/custom-product.css` — companion CSS
- Template JSON `order`: `["custom_product_XXXXX"]` — ONLY this section
- `main-product` MUST NOT appear in the template order array
- Gallery: 55% width (flex: 0 0 55%), Buy box: 45%
- Verify by re-reading the template JSON after upload
- **Confirm `main-product` / `product-information` are NOT in the order array** — if they are, remove them and re-upload immediately
- If you find yourself putting buy box content (benefits, FAQ, reviews) as separate page sections → STOP, you are doing it wrong, move them inside custom-product.liquid as blocks

**Mobile optimization (mandatory):**
- On mobile (≤749px): gallery stacks above buy box, both 100% width
- Thumbnails: horizontal scroll row (`overflow-x: auto`, `flex-direction: row`)
- ATC button: `width: 100%`, `min-height: 52px`
- Title: `font-size: 22px` on mobile
- Benefit pills: `flex-wrap: wrap`, smaller padding
- Trust badges: center-aligned, smaller icons

**Icons in buy box:** `ICON_STYLE` from Step 0.5. If SVG → use SVG library from `references/above-the-fold.md`. Never emoji.

**Colors in buy box:** Use ONLY the competitor's exact colors from Step 1 analysis. Never use defaults. ATC button = competitor's CTA button color exactly.

**Drag & drop:** The buy box uses Shopify native blocks. User reorders them in the theme customizer. Katching Bundles installs as an app block between pills and ATC.

### 6b — Below-fold sections

Read `references/landing-designer.md`, `references/elixir-design-language.md`, `references/section-schema-standards.md`.

**SECTION ORDER IS MANDATORY — match competitor exactly:**
1. Number every section in the competitor PDF top to bottom: 1, 2, 3, 4...
2. Build them in that exact order. Do not reorder, skip, or add sections not in the competitor.
3. Before building each section, write one line describing what it is in the competitor: "Section 3: horizontal photo carousel with 8+ customer photos, no videos, names below each photo"
4. Build THAT section — not your interpretation of it.
5. Name each .liquid file with its number: `section-01-benefits.liquid`, `section-02-stats.liquid`, etc.
6. Add them to the template JSON `order` array in the same numbered order.

**BRAND NAME RULE:** Replace every instance of the competitor's brand name with PRODUCT_TITLE or the user's brand. Search-replace before finalizing any section.

One `.liquid` file per section. Never combine.

**Every section must have:**
- `{% style %}` block with `#section-{{ section.id }}` CSS variables
- Two-part accent heading (regular + accent color)
- `use_theme_colors` toggle
- Background type: solid/gradient/image
- Padding: 4-side desktop + 4-side mobile (8 settings)
- Section borders: top + bottom independently with style selector
- All typography: desktop size + mobile size separately
- Animation setting — **default to `slide-up` for all sections** (not `none`)
- Google Font — **always load and apply** the font that best matches competitor's heading style
- Mobile-first CSS

**Anti-React checklist (apply to every section):**
- Load a Google Font for headings — never use system-ui or generic sans-serif alone
- Add `transition` on hover states for all interactive elements
- Use `letter-spacing` on headings (0.5-2px typically)
- Cards must have `box-shadow` or `border` — never flat white boxes with no definition
- Section backgrounds must alternate: white → tinted (e.g. #f8faf9) → dark → white
- Accent text color must be different from body text — always use `COLOR_PRIMARY`

**Section quality checklist (25 items) — read `references/section-schema-standards.md`**

### 6c — Higgsfield images Phase B ⚠️ RULE #5

**Run immediately after 6b. Do NOT skip.**

1. List every image slot in every section .liquid file
2. For each slot: identify what type of image the competitor has in that position
3. Generate matching image in Higgsfield (model: `nano_banana_2`)
4. Upload to Shopify Files → get CDN URL
5. Inject URL into section .liquid file → re-read to confirm

No grey boxes allowed. Every slot must have a real image before proceeding to Step 6.5.

---

## Step 6.5 — Global theme ⚠️ RULES #3 #4

**Run after 6c. All 3 sub-steps are mandatory.**

### Announcement bar ⚠️ RULE #4

**The announcement bar must replicate the competitor's exactly — adapted to the brand's colors, language, and product name. No preset rules. Read the competitor PDF and copy what they have.**

1. From ANNOUNCEMENT_ANALYSIS, extract:
   - How many bars? (1 or 2)
   - Exact text content of each bar (translated to OUTPUT_LANGUAGE, adapted to this product)
   - Bar type: static / scrolling ticker / countdown / combination
   - Exact background color, text color, font weight
   - Divider style between items (pipe, dot, star, none)
   - Icon style per item (SVG — never emoji unless competitor uses emoji)
   - Position (above header / below header)

2. Read `sections/announcement-bar.liquid` → check if `richtext` or plain `text` field
3. Plain text → create `sections/custom-announcement-bar.liquid` from scratch with full HTML
4. Comment out `{% section 'announcement-bar' %}` in `layout/theme.liquid`
5. Add custom bar to `layout/theme.liquid` in the correct position
6. Re-read both files after writing to confirm saved
7. **Verify:** re-read `layout/theme.liquid` — native bar must be commented out, custom bar must be present

### Header

1. Update `config/settings_data.json` — colors, sticky, logo
2. If image logo needed → ask user for upload
3. Cart icon → match competitor via settings or CSS override
4. Re-read settings_data.json to confirm

### Footer ⚠️ RULE #3

1. Create `sections/custom-footer.liquid` from scratch
2. Comment out `{% section 'footer' %}` in `layout/theme.liquid`
3. Add `{% section 'custom-footer' %}` in its place
4. Populate from `FOOTER_ANALYSIS`:
   - Brand column (logo or text name + description)
   - Link columns (Menu, Contact, Policies — match competitor)
   - Payment icons (native `shop.enabled_payment_types`)
   - Copyright text
   - Legal disclaimer if competitor has one
5. Create `assets/custom-footer.css` — mobile-first
6. Re-read `layout/theme.liquid` to confirm native footer is commented out

**FAIL CONDITIONS — fix before QA:**
- Native footer still visible (not commented out)
- "My Store 3" text anywhere
- "Join our email list" in English
- "Tecnología de Shopify" visible

### ATC button color

Read `config/settings_data.json` — find button color settings. Update there.
If not available: add to `assets/custom.css`:
```css
button[name="add"], .cp-atc {
  background-color: COLOR_PRIMARY !important;
  color: #ffffff !important;
}
```
Confirm `custom.css` is linked in `layout/theme.liquid`.

---

## Step 7 — QA with 3-pass visual verification

Read `references/qa-checklist.md`.

**3-pass verification against competitor — mandatory before delivery:**

### Pass 1 — Structure check
Open the live landing preview URL in Chrome. Compare against the competitor PDF:
- [ ] Same number of sections in same order
- [ ] Buy box elements match competitor's buy box elements
- [ ] Announcement bar matches competitor's type and content
- [ ] Header style matches competitor
- [ ] Footer structure matches competitor

Fix any structural differences before Pass 2.

### Pass 2 — Visual/branding check
Still comparing in Chrome:
- [ ] Colors match exactly (primary, backgrounds, text, CTAs)
- [ ] Font style matches (serif/sans, weight, size ratio)
- [ ] Section spacing/padding feels the same
- [ ] Cards have correct border-radius and shadow
- [ ] Icons are SVG not emoji (unless competitor uses emoji)
- [ ] No grey placeholder boxes
- [ ] No "My Store 3" or untranslated English text

Fix any visual differences before Pass 3.

### Pass 3 — Mobile check
Resize Chrome to 390px width. Compare both pages on mobile:
- [ ] Gallery stacks above buy box correctly
- [ ] ATC button full width, easy to tap
- [ ] Text readable without zooming (min 14px)
- [ ] No horizontal scroll
- [ ] Sections stack correctly

Only deliver after all 3 passes confirm match.

**If any pass fails:** fix the issue, then redo ALL 3 passes from the start.

---

## Step 8 — Higgsfield product photos (Phase A) — AUTOMATIC, no confirmation needed

Run immediately after Step 7 QA. Do NOT wait for user approval. Do NOT ask for confirmation.

**Product photos use PRODUCT_PHOTOS_FOLDER (local desktop folder).**
Read each reference image from that folder using the filesystem tool.
Replicate each composition using PRODUCT_PHOTO as the product reference.
See `references/higgsfield-photos.md` Step A1-A4 for full workflow.

**Landing section images (Phase B):** extracted from competitor PDF and Chrome analysis.
No Drive, no folder. Claude identifies what each section's image should look like from the competitor,
generates it in Higgsfield, and injects it into the section file.

Read `references/higgsfield-photos.md` for full workflow.

**Automatic sequence:**
1. Generate 4-6 product images using PRODUCT_PHOTO as reference (lifestyle + packshot)
2. Upload all generated images to the Shopify product gallery via Files API
3. Set the first image as the featured product image
4. Run Phase B (landing section images) if not already done
5. Print final output summary

No intermediate confirmation needed. Complete all steps automatically.

---

## Batch mode

If prompt contains multiple `COMPETITOR_URL` entries or a `LANDINGS:` block:
- Process each landing fully (Steps 1–8) before starting the next
- Mode A: always duplicate from the same `BASE_THEME_ID` for each landing
- Auto-continue between landings, no confirmation needed
- Print batch summary at end

---

## Theme modes for above-the-fold

⚠️ ALL THREE TYPES end up with `custom-product.liquid` as the primary section.
The type only determines whether any native section stays in the template at all.

Read `references/above-the-fold.md` Step 1 to detect:
- **Type A** (Elixir block-based) → can optionally enhance native blocks, OR use custom-product
- **Type B** (Dawn with custom_liquid) → can inject HTML blocks, OR use custom-product
- **Type C** (Horizon, any theme without custom_liquid) → ALWAYS use custom-product

**DEFAULT FOR ALL THEMES:** Use `custom-product.liquid`. It works on every theme.
**NEVER** keep `product-information`, `main-product`, or any native buy box in the template order.
**NEVER** put buy box content in sections below the fold.

---

## Final output

```
========================================
  LANDING PAGE COMPLETE
========================================
Product:   PRODUCT_TITLE
Template:  product.TEMPLATE_NAME
Theme:     WORKING_THEME_ID
Price:     PRODUCT_PRICE

Preview:   https://STORE_URL/products/PRODUCT_HANDLE
Admin:     https://STORE_URL/admin/themes/WORKING_THEME_ID/editor
========================================
Next: Add Katching Bundles app block in theme customizer
      between Benefit Pills and ATC button in Custom Product section
========================================
```
