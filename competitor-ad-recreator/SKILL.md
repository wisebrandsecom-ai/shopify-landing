# Competitor Ad Recreator — Static Ads

Recreates competitor static ads using a local folder of reference images + optional copy file.
Adapts visual structure and copy to the user's product, branding, and benefits.

---

## ⛔ HARDCODED RULES

1. **READ EVERY IMAGE** — Never skip an image in ADS_FOLDER. Read each one visually before writing any prompt. Print an inventory table first.
2. **NO PLACEHOLDERS** — Every prompt field must be filled. Forbidden: "clean background", "nice composition", "beautiful product". Be specific.
3. **PRINT-BEFORE-ACT** — Print the Pre-flight Checklist before any generate_image call.
4. **ONE IMAGE = ONE GENERATION** — N ad images in folder = N Higgsfield generations. No combining, no skipping.
5. **FAIL LOUD** — If a generation doesn't match the reference structure, print a DIVERGENCE REPORT and stop. Do not upload bad results.
6. **BRAND IS USER'S, STRUCTURE IS COMPETITOR'S**

7. **LOCALE-AWARE COPY** — Copy must feel native to OUTPUT_LOCALE, not like a literal translation.
   - Use vocabulary natural to that country (e.g. ES/MX "coche" vs "carro", "vosotros" never in MX)
   - Adapt idioms, expressions, and urgency language to local register
   - Prices, units, and references must match local conventions
   - Tone stays the same as the reference ad — do not add or remove formality
   - Never exaggerate regional accent or slang — write how a native speaker would, not a caricature
   - When OUTPUT_LOCALE is not provided: infer from BRAND_NAME context or default to neutral Spanish — Borrow the visual layout. Replace everything identity-related with the user's brand.

7. **NATIVE vs STATIC DETECTION** — Classify each ad image before generating:
   - **STATIC**: product is visible in the image → `nano_banana_2` + PRODUCT_MEDIA_ID
   - **NATIVE**: product NOT visible, looks like real person/UGC/lifestyle → `soul_2` (no Soul ID needed)
   Detect automatically from visual analysis in Block 1. If ambiguous → default to STATIC.

---

## Required inputs

```
AD_SOURCE         meta_library_url OR local_folder
                  - meta_library_url: URL of advertiser's Meta Ad Library profile
                  - local_folder: /Users/.../Desktop/my-ads-folder
                    Each subfolder: one image + optional copy.txt

PRODUCT_PHOTO     [attached PNG — clean packshot, white/transparent bg]
PRODUCT_DOC       [attached PDF — product benefits, tone, features]
AVATAR_DOC        [attached PDF — only required for Copy Mode C]
COPY_MODE         A / B / C  (see below)
OUTPUT_LANGUAGE   es / en / fr / etc.
OUTPUT_LOCALE     [country code — e.g. MX, ES, CO, AR, US]
                  Determines regional vocabulary, expressions, and cultural references.
                  Required when OUTPUT_LANGUAGE = es
OUTPUT_FOLDER     [local path — e.g. /Users/.../Desktop/Batch 001 Output] or pt.
BRAND_NAME        [brand name as it appears on packaging]
BRAND_COLORS      [primary hex — e.g. #8B2C2C]
```

If any input is missing → ask before starting. Do not assume.

---

## Copy modes

### Mode A — 1:1 Copy
Reproduce the competitor's copy as-is.
- Keep same structure, emotional angle, CTA style
- Translate to OUTPUT_LANGUAGE if different, localized to OUTPUT_LOCALE
- Only swap brand/product name where necessary

### Mode B — Translate + adapt
Light adaptation:
- Translate to OUTPUT_LANGUAGE, localized to OUTPUT_LOCALE
- Replace competitor brand/product with user's product
- Keep creative strategy and emotional angle intact
- Adapt idioms and expressions to feel native in OUTPUT_LOCALE
- Draw benefits from `brand_profile` where substitution is needed

### Mode C — Research rewrite
Full rewrite using competitor structure as inspiration only.
- **Requires**: AVATAR_DOC
- Borrow the emotional hook and format from the competitor
- Write entirely new copy rooted in user's avatar + product
- Lead with avatar's massive desire or main pain point
- Match awareness level from AVATAR_DOC

---

## Execution blocks

### BLOCK 0 — Read inputs + inventory

1. Upload PRODUCT_PHOTO to Higgsfield → store as `PRODUCT_MEDIA_ID`
2. Read PRODUCT_DOC → extract `brand_profile`:
   - Core offer (1 sentence)
   - Top 5 benefits (flag most visual ones)
   - Tone of voice
   - Visual identity cues (colors, style)
   - Target audience
3. If COPY_MODE = C → read AVATAR_DOC → extract `avatar_profile`:
   - Massive desire
   - Top 3-5 pain points
   - Awareness level
   - Demographics
   - Language patterns (exact phrases)
4. Read ads based on AD_SOURCE:

   **If AD_SOURCE = meta_library_url:**
   Use Claude in Chrome (Claude_in_Chrome MCP) — no Gemini, no web_fetch, no API keys.
   Read `references/meta-library-scraper.md` and execute it fully.
   Result: `ads[]` array with image URL + copy text per ad.

   **If AD_SOURCE = local_folder:**
   List subfolders. For each:
   - Read the image file → `ad[n].image`
   - Read copy.txt if present → `ad[n].copy` (null if missing)
   
   Print inventory: N ads found = N generations planned.

Show summary to user. Confirm before Block 1.

---

### BLOCK 1 — Visual analysis (HARD GATE)

Read each ad image visually (from local file or downloaded from Meta). Print this table before any generation:

```
| # | Filename | **Type** | Layout | Subject | Background | Text overlays (source) | Aspect | Mood |
|---|----------|----------|--------|---------|------------|------------------------|--------|------|
| 1 | ad1.jpg  | **STATIC** | split | product left, text right | white gradient | "LOSE 10KG IN 30 DAYS" top | 1:1 | urgent |
| 2 | ad2.jpg  | **NATIVE** | full-bleed | woman in kitchen, no product | warm home | none | 4:5 | warm |

Type detection rule:
- STATIC = product packaging/bottle/bag visible anywhere in the image
- NATIVE = no product visible, looks like organic social content (person, lifestyle, UGC)
```

Count total: `N images found → N generations planned`

Each ad has its own copy.txt in its subfolder — use `ad[n].copy` directly. No matching needed.

---

### BLOCK 2 — Copy generation

For each ad image, generate adapted copy using COPY_MODE.

Also produce two fields:
- `TEXT_OVERLAYS_SOURCE` — verbatim text visible in the reference image
- `TEXT_OVERLAYS_OUTPUT_LANG` — same text in OUTPUT_LANGUAGE (goes in Higgsfield prompt)

**Show all adapted copy to user before Block 3. User confirms or edits.**

```
AD 1 — [filename]
  Headline:      [adapted]
  Primary text:  [adapted]
  CTA:           [adapted]
  Overlay (source lang):  [verbatim from image]
  Overlay (OUTPUT_LANG):  [translated]
```

---

### BLOCK 3 — Image prompt construction

Read `references/higgsfield-prompting.md` before this block.

For each ad, fill this strict template:

```
REFERENCE_FILE:         [filename]
LAYOUT:                 [exact layout: split / centered / top-bottom / etc.]
SUBJECT:                [what is the main visual subject and where]
BACKGROUND:             [exact color + material — "warm cream linen" not "clean background"]
LIGHTING:               [direction, quality, color temp]
PROPS_EXACT:            [every visual element: list them all]
TEXT_OVERLAYS:          [TEXT_OVERLAYS_OUTPUT_LANG content, position, style]
BRAND_TREATMENT:        ["product packaging shows '[BRAND_NAME]', colors [BRAND_COLORS]"]
MOOD:                   [urgent / aspirational / clinical / warm / bold / etc.]
ASPECT_RATIO:           [match reference]
```

Forbidden in final prompt: "beautiful", "amazing", "nice", "clean background" alone, "minimal composition" alone.

**Pre-flight checklist (print before calling generate_image):**

```
PRE-FLIGHT [ad N]:
[ ] Inventory table printed
[ ] generations_planned = N == image_files_in_folder = N
[ ] TEXT_OVERLAYS_OUTPUT_LANG verified in OUTPUT_LANGUAGE (zero source-language words)
[ ] PRODUCT_MEDIA_ID will be passed in medias[]
[ ] All prompt fields filled — no placeholder phrases
[ ] BRAND_COLORS in prompt
```

---

### BLOCK 4 — Generate in Higgsfield

Model depends on ad type from Block 1:

**STATIC ads** → `nano_banana_2`
```
Model: nano_banana_2
Medias: [{ value: PRODUCT_MEDIA_ID, role: "image" }]
Prompt: [from Block 3 template]
Aspect ratio: [from Block 3]
```

**NATIVE ads** → `soul_2`
```
Model: soul_2
Prompt: [UGC-style prompt from references/higgsfield-prompting.md Soul 2.0 template]
Aspect ratio: [from Block 3]
```
No Soul ID needed. `soul_2` generates realistic people from a text description.
Describe the person in the prompt: age, appearance, clothing, setting, action, emotional tone.

After each generation, print verification block:

```
VERIFICATION [filename]:
  Layout matches reference?         [YES/NO]
  Brand/product visible?            [YES/NO]
  Overlays in OUTPUT_LANGUAGE?      [YES/NO — list any wrong-language words]
  → PASS (upload) | FAIL (retry once) | DIVERGENCE (stop + report)
```

If FAIL → adjust only the failing element, retry once.
If second attempt fails → print DIVERGENCE REPORT, wait for user.

---

### BLOCK 5 — Save to desktop

```bash
mkdir -p "OUTPUT_FOLDER"
curl -L "HIGGSFIELD_CDN_URL_1" -o "OUTPUT_FOLDER/anuncio_01.png"
curl -L "HIGGSFIELD_CDN_URL_2" -o "OUTPUT_FOLDER/anuncio_02.png"
```

Write one `copy_anuncio_0N.txt` per ad. N ads = N files. Never combine.
Format: see `references/copy-format.md`.

```
OUTPUT_FOLDER/
  anuncio_01.png
  copy_anuncio_01.txt
  anuncio_02.png
  copy_anuncio_02.txt
```

Print OUTPUT_FOLDER path and finish. No Drive. No upload. Done.
---

## Error handling

| Error | Action |
|-------|--------|
| Image unreadable by Read tool | Ask user to re-export as PNG/JPG |
| copies.txt not found | Skip — generate copy from image analysis only |
| Higgsfield fails twice | Skip ad, note in copies.txt, continue |
| PRODUCT_MEDIA_ID fails | Stop Block 4, ask user to re-attach product photo |

| Avatar PDF missing in Mode C | Stop Block 2, request before continuing |
