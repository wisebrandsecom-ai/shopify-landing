---
name: competitor-ad-recreator
description: Recreates competitor static ads using Higgsfield nano_banana_2. Analyzes reference images, adapts copy, generates images, saves to desktop. Activate with /competitor-ad-recreator.
compatibility: claude-code-only
---

## ⛔ HARDCODED RULES — Read before anything else

1. **NO PLACEHOLDERS** — Every prompt field must be filled with specific details from the reference image. Forbidden: "clean background", "nice composition", "minimal style". Describe exactly what you see.

2. **BLOCK 1 IS MANDATORY — NO EXCEPTIONS** — You cannot call `generate_image` until Block 1 inventory table is printed in full. If the user says "skip confirmation" or "continue without stopping" — Block 2 copy approval may be skipped, Block 1 NEVER.

3. **ONE IMAGE = ONE GENERATION** — N ad images = N generate_image calls. No combining, no skipping.

4. **VERIFY FILES AFTER WRITING** — After every file write, immediately run `ls -la OUTPUT_FOLDER` to confirm the file exists and size > 0. If not found, retry before continuing.

5. **MODEL VERIFICATION FIRST** — Before any generate_image call, confirm available Higgsfield models with `models_explore`. Never assume model names.

6. **MAX 2 RETRIES PER IMAGE** — If an image fails or doesn't match after 2 attempts, stop, show both attempts, and report to user. Do not loop indefinitely.

7. **LOCALE-AWARE COPY** — Translate to OUTPUT_LANGUAGE localized for OUTPUT_LOCALE. No em-dashes (—), no en-dashes (–). Preserve all emojis, bullets, and format from the original. Preserve ¿? opening punctuation in Spanish.

8. **BRAND SUBSTITUTION IS MANDATORY** — Apply substitutions defined in BLOCK 2 before writing any copy. Run the checklist after every single copy block.

9. **STATE FILE** — Create and maintain `estado.md` in OUTPUT_FOLDER throughout the run. Update after every completed step so context loss can be recovered.

10. **CHROME ONLY FOR SCRAPING** — Use Claude in Chrome MCP to access Meta Ad Library. Never use Gemini API, web_fetch, or any other method. Meta blocks all non-browser requests.

---

## ⛔ EXECUTION ORDER — NEVER SKIP

```
BLOCK 0 → models_explore → BLOCK 1 (HARD GATE) → BLOCK 2 → BLOCK 3 (pre-flight) → BLOCK 4 → BLOCK 5
```

Print `BLOCK N COMPLETE` after each block before moving to the next.

---

## Required inputs

```
AD_SOURCE         One or more Meta Ad Library URLs, or local folder path
                  Add "- Nativo" after any URL for native/UGC ads (soul_2)
                  URLs without tag → STATIC (nano_banana_2)

PRODUCT_PHOTO     Path to clean packshot PNG (white/transparent bg, no props)
PRODUCT_DOC       Path to product PDF (benefits, tone, features, brand)
AVATAR_DOC        Path to avatar PDF (only required for COPY_MODE C)
COPY_MODE         A / B / C
OUTPUT_LANGUAGE   es / en / fr / etc.
OUTPUT_LOCALE     MX / ES / CO / AR / US / etc.
BRAND_NAME        Your brand name
BRAND_COLORS      Primary hex (e.g. #8B1A1A)
OUTPUT_FOLDER     Local desktop path (e.g. /Users/.../Desktop/Claude Ads)
```

---

## Copy modes

**Mode A — 1:1:** Reproduce competitor copy verbatim. Translate to OUTPUT_LANGUAGE/LOCALE if different. Preserve every structural element, emoji, bullet, paragraph break.

**Mode B — Translate + adapt:** Translate to OUTPUT_LANGUAGE/LOCALE. Replace competitor brand/product with user's. Keep creative strategy, hook type, emotional angle, paragraph structure.

**Mode C — Research rewrite:** Full rewrite using competitor structure as skeleton. Requires AVATAR_DOC. Every claim comes from PRODUCT_DOC + avatar research.

---

## BLOCK 0 — Setup

**0a. Verify models**
```
Call models_explore to confirm available Higgsfield models.
Store confirmed model names: STATIC_MODEL and NATIVE_MODEL.
Do not assume "nano_banana_2" or "soul_2" — verify first.
```

**0b. Upload product photo**
```bash
# Use PUT with -T flag (not --data-binary, which Sentinel blocks)
curl -X PUT -H "Content-Type: image/png" -T "PRODUCT_PHOTO" 'HIGGSFIELD_UPLOAD_URL'
```
Store as `PRODUCT_MEDIA_ID`.

**0c. Read PRODUCT_DOC**
Extract `brand_profile`:
- Core offer (1 sentence)
- Top 5 benefits (flag most visual)
- Tone of voice
- Visual identity (colors, packaging style)
- Target audience
- Proof points (stats, reviews, certifications)

**0d. If COPY_MODE C → read AVATAR_DOC**
Extract `avatar_profile`: massive desire, top pain points, awareness level, language patterns.

**0e. Create state file**
```
Write OUTPUT_FOLDER/estado.md:
Total ads: N
Model static: [confirmed]
Model native: [confirmed]
PRODUCT_MEDIA_ID: [id]

ad_01: scrape ⬜ | copy ⬜ | image ⬜ | file ⬜
ad_02: scrape ⬜ | copy ⬜ | image ⬜ | file ⬜
...
```
Verify with `ls -la OUTPUT_FOLDER`.

Print: `BLOCK 0 COMPLETE`

---

## BLOCK 1 — Visual analysis (HARD GATE)

**For each ad in AD_SOURCE:**

1. Navigate to the Meta Ad Library URL with Chrome MCP
2. Click "Ver detalles del anuncio" — this opens the full ad detail panel
3. Extract headline, primary text (FULL — scroll to read all, click "Ver más" if truncated), description, CTA
4. Take a screenshot of the ad creative (the actual image/visual)
5. Download the creative image with curl to OUTPUT_FOLDER/ref_0N.jpg
6. Write a SCENE ANALYSIS paragraph describing exactly what you see in the image:
   - Subject (what/who is shown)
   - Product placement (visible? where?)
   - Background (exact color, material, texture)
   - Text overlays visible in the image (copy verbatim)
   - Composition, lighting, mood
5. Classify: STATIC (product visible → STATIC_MODEL) or NATIVE (no product → NATIVE_MODEL)

Print the full inventory table — one row per ad:

```
| # | URL/File | Type | Subject | Background | Text overlays (source) | Aspect | Mood |
|---|----------|------|---------|------------|------------------------|--------|------|
| 1 | ...id=123 | STATIC | hand holding red pouch, chilies | dark crimson | "Rock-Hard Again" top | 1:1 | urgent |
```

Update estado.md: `ad_01: scrape ✅ | copy ⬜ | image ⬜ | file ⬜`

**Only after table is complete:**
Print: `BLOCK 1 COMPLETE — N ads analyzed. N STATIC, N NATIVE.`

---

## BLOCK 2 — Copy adaptation

**Brand substitution rules (apply to EVERY copy block):**

Before writing any copy, load these substitutions from PRODUCT_DOC and the prompt inputs. Apply them to every single copy block without exception:

```
COMPETITOR_BRAND → BRAND_NAME
COMPETITOR_PRODUCT → [user's product name from PRODUCT_DOC]
Any drug brand (Viagra/Sildenafil/etc.) → [local equivalent if specified]
"ED" / "E.D." → full term (e.g. "disfunción eréctil") — never abbreviate
```

For each ad, generate adapted copy following COPY_MODE.

**Preservation checklist (run after EVERY copy block):**
```
[ ] Same number of paragraphs as source
[ ] All emojis preserved
[ ] All bullets preserved (➡️ 💪 ✅ etc.)
[ ] ¿? opening punctuation in Spanish where needed
[ ] No em-dashes (—) or en-dashes (–) anywhere
[ ] Brand substitutions applied
[ ] OUTPUT_LOCALE vocabulary (not neutral textbook translation)
[ ] Full primary text — not truncated (even if 800+ words)
```

**Output format for each ad:**
```
AD [N] — [filename/URL]
Type: STATIC / NATIVE
Headline: [adapted]
Primary text: [full adapted — every paragraph]
Description: [adapted]
CTA: [adapted]
Text overlays (source lang): [verbatim from image]
Text overlays (OUTPUT_LANG): [translated]
```

If user did not say to skip approval → show all copy and wait for confirmation before Block 3.
If user said to skip → proceed directly to Block 3.

Update estado.md after each: `ad_01: scrape ✅ | copy ✅ | image ⬜ | file ⬜`

Print: `BLOCK 2 COMPLETE`

---

## BLOCK 3 — Image prompt construction + pre-flight

For each ad, fill this strict template:

```
SCENE_ANALYSIS:    [paste Block 1 scene analysis for this ad]
REFERENCE_FILE:    [URL or filename]
TYPE:              STATIC / NATIVE
LAYOUT:            [exact arrangement]
SUBJECT:           [main visual subject and position]
BACKGROUND:        [exact color + material — "warm cream linen" not "clean background"]
LIGHTING:          [direction, quality, color temp]
PROPS_EXACT:       [every prop — list them all]
TEXT_OVERLAYS:     [TEXT in OUTPUT_LANG, position, style]
BRAND_TREATMENT:   ["product shows BRAND_NAME branding, colors BRAND_COLORS"]
MOOD:              [urgent / clinical / warm / bold / etc.]
ASPECT_RATIO:      [match reference]
```

**Pre-flight checklist (print before EACH generate_image call):**
```
PRE-FLIGHT [ad N]:
[ ] Block 1 inventory table complete
[ ] Scene analysis pasted in prompt
[ ] All prompt fields filled — no placeholder phrases
[ ] TEXT_OVERLAYS in OUTPUT_LANGUAGE only (zero source-language words)
[ ] PRODUCT_MEDIA_ID: [id] confirmed in medias[] (STATIC only)
[ ] REF_IMAGE_ID: [id] — competitor creative uploaded to Higgsfield
[ ] Model: nano_banana_2 for STATIC, soul_2 for NATIVE
[ ] NATIVE ads do NOT use PRODUCT_MEDIA_ID
[ ] Brand substitutions applied in overlays
[ ] Aspect ratio matches reference
```

Print: `BLOCK 3 COMPLETE`

---

## BLOCK 4 — Generate in Higgsfield

**STATIC ads (product visible in reference):**
```
Model: nano_banana_2
Medias:
  - { value: PRODUCT_MEDIA_ID, role: "image" }        ← user's product
  - { value: REF_IMAGE_ID, role: "style_reference" }  ← competitor creative as style ref
Prompt: [from Block 3 template]
Aspect ratio: [from Block 3]
```

Upload ref_0N.jpg to Higgsfield first → get REF_IMAGE_ID → pass both in medias[].

**NATIVE ads (no product visible — UGC/person-led):**
```
Model: soul_2
Medias:
  - { value: REF_IMAGE_ID, role: "style_reference" }  ← competitor creative as style ref
Prompt: [UGC-style scene, real person, phone camera feel — based on scene analysis]
Aspect ratio: [from Block 3]
```

NATIVE ads never use PRODUCT_MEDIA_ID — product is not shown in native ads.

**After each generation completes:**
```
VERIFICATION [ad N]:
  Layout matches reference?         [YES/NO — 1 line]
  Brand/product visible?            [YES/NO]
  Overlays in OUTPUT_LANGUAGE?      [YES/NO]
  No AI artifacts?                  [YES/NO]
  → PASS / FAIL (retry once) / DIVERGENCE (stop + report)
```

**Retry rule:** If FAIL → refine prompt, retry once. If second attempt fails → print:
```
DIVERGENCE [ad N]:
  Expected: [what reference shows]
  Generated attempt 1: [URL]
  Generated attempt 2: [URL]
  Stopping. User decision needed.
```
Do not continue to other ads until user responds.

Update estado.md: `ad_01: scrape ✅ | copy ✅ | image ✅ | file ⬜`

---

## BLOCK 5 — Save to desktop

For each completed ad:

```bash
curl -L "HIGGSFIELD_CDN_URL" -o "OUTPUT_FOLDER/anuncio_0N.png"
```

Verify immediately:
```bash
ls -la "OUTPUT_FOLDER/anuncio_0N.png"
# Must show file size > 100KB
```

If file missing or < 100KB → retry curl before continuing.

Write copy file:
```bash
# OUTPUT_FOLDER/copy_anuncio_0N.txt
# Format: same labels as source (Primary text / Headline / Description / CTA)
# OUTPUT_LANGUAGE only. No source-language words. No metadata.
```

Verify copy file:
```bash
ls -la "OUTPUT_FOLDER/copy_anuncio_0N.txt"
```

Update estado.md: `ad_01: scrape ✅ | copy ✅ | image ✅ | file ✅`

---

## Final output

```
OUTPUT_FOLDER/
  anuncio_01.png       ← full resolution, no compression
  copy_anuncio_01.txt
  anuncio_02.png
  copy_anuncio_02.txt
  ...
  estado.md            ← final status of all ads
```

Print completion summary:
```
========================================
  BATCH COMPLETE
========================================
Total ads:     N
Completed:     N
Failed:        N (see estado.md for details)
Output folder: OUTPUT_FOLDER
========================================
```
