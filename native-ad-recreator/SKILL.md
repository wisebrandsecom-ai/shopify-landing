# Native Ad Recreator

Recreates competitor native ads using Higgsfield Higgsfield Soul (Realistic).
Adapts copy with full preservation of structure, tone, and marketing strategy.
Native ads = no product visible, looks like a real person's organic post.

---

## ⛔ HARDCODED RULES

1. **SOUL 1.0 ONLY** — Always use `soul_2` with `style: realistic`. Never nano_banana_2 for this skill.

2. **GENERATE 3, PICK 1** — Generate 3 variations per ad. Pick the one that looks most like a real person took it. Reject any image with: extra limbs, distorted faces, exaggerated expressions, weird angles, obvious AI artifacts, anything that looks "generated".

3. **COPY FIDELITY** — The copy is the most important asset. Rules:
   - **FULL LENGTH MANDATORY** — Storytelling copy is long by design. Output every paragraph. Never summarize, truncate, or cut for length. If the original is 800 words, output is ~800 words.
   - Preserve exact structure (paragraph breaks, sentence rhythm, list format)
   - Preserve marketing strategy (hook, problem, agitation, solution, CTA flow)
   - Preserve tone (urgency, intimacy, authority — whatever the original has)
   - Never add content that isn't in the original
   - Never remove content from the original
   - Never "improve" the copy — your job is to translate/adapt, not to edit
   - No em dashes, no hyphens between phrases, no bullet points unless original has them
   - Count paragraphs in source. Output must have same paragraph count. Verify before saving.

4. **NO GUIONES** — Em dashes (—), hyphens used as separators (-), and en dashes (–) are forbidden in copy output. If the original uses them as separators, replace with a period or comma.

5. **LOCALE-AWARE** — Translate to OUTPUT_LANGUAGE localized for OUTPUT_LOCALE. Natural native register. No exaggerated slang. No literal word-for-word translation that sounds foreign.

6. **ONE FILE PER AD** — `copy_nativa_0N.txt` for each ad. Never combine.

7. **PRINT-BEFORE-ACT** — Print pre-flight checklist before any generate_image call.

---

## Required inputs

```
AD_SOURCE         meta_library_url OR local_folder
PRODUCT_DOC       [PDF with product info — required for COPY_MODE B]
AVATAR_DOC        [PDF with avatar/customer profile — optional, strengthens Mode B]
COPY_MODE         A / B  (see below)
OUTPUT_LANGUAGE   es / en / fr / etc.
OUTPUT_LOCALE     MX / ES / CO / AR / US / etc.
BRAND_NAME        [your brand name]
OUTPUT_FOLDER     [local path — e.g. /Users/.../Desktop/Nativas Output]
```

---

## Copy modes

### Mode A — Translate + preserve
Translate the competitor's copy to OUTPUT_LANGUAGE/OUTPUT_LOCALE.
- Preserve every structural element: paragraph order, sentence count, hook placement, CTA position
- Preserve marketing strategy: hook type, problem framing, social proof placement, urgency mechanism
- Preserve tone: if it's intimate → keep intimate. If urgent → keep urgent. If conversational → keep conversational
- Only change: language, brand/product name if mentioned
- Do NOT: summarize, improve, add transitions, restructure, modernize phrasing

### Mode B — Recreate with your product
Use competitor copy structure as a blueprint. Rewrite for your product using PRODUCT_DOC.
- Extract the structural skeleton: hook type, problem description pattern, solution reveal, proof type, CTA
- Rebuild each section with your product's story, benefits, and customer language from AVATAR_DOC
- Match length: if original is 400 words, output is ~400 words
- Match paragraph count and rhythm
- Never borrow specific claims from competitor — only the architecture

---

## Execution blocks

### BLOCK 0 — Read inputs

1. Read PRODUCT_DOC → extract brand_profile (offer, benefits, tone, audience)
2. If COPY_MODE B and AVATAR_DOC provided → extract avatar_profile (pain, desire, language patterns)
3. Read ads based on AD_SOURCE:
   - `meta_library_url` → use Claude in Chrome (read `references/meta-library-scraper.md`)
   - `local_folder` → read image + copy.txt from each subfolder

Print: N ads found = N generations planned

---

### BLOCK 1 — Visual analysis (HARD GATE)

For each native ad image, print this table before any generation:

```
| # | Source | Scene | Person (age/gender/ethnicity) | Setting | Lighting | Angle | Expression | Mood | Aspect |
|---|--------|-------|-------------------------------|---------|----------|-------|------------|------|--------|
| 1 | ad1.jpg | woman sitting at kitchen table, phone in hand | F, ~45, caucasian | warm home kitchen, morning | soft window light left | eye-level, slight angle | relaxed, candid | organic UGC | 1:1 |
```

generations_planned = N == ads_found = N ✅

---

### BLOCK 2 — Copy adaptation

For each ad, adapt the copy using COPY_MODE.

**Before writing:** extract the structural skeleton of the original:
```
SKELETON [ad N]:
  Hook type:        [curiosity / pain / statement / question / story opening]
  Hook length:      [N words]
  Problem section:  [N paragraphs, framing type]
  Agitation:        [yes/no — emotional amplification pattern]
  Solution reveal:  [position in copy, how introduced]
  Proof:            [type: statistic / testimonial / personal story / none]
  CTA:              [style and position]
  Total length:     [N words]
  Paragraph count:  [N]
```

Then write the adapted copy matching that skeleton exactly.

Show copy to user for approval before Block 3. User confirms or edits.

---

### BLOCK 3 — Image prompt construction

For each ad, read `references/soul-prompting.md` and write a prompt for Higgsfield Soul.

The prompt must describe a scene that a real person could have taken with their phone:
- Natural setting (home, street, park, café)
- Real person with believable age/appearance
- Candid or semi-candid composition
- No studio lighting, no professional photography feel
- Phone-camera perspective

```
PRE-FLIGHT [nativa N]:
[ ] Inventory table printed
[ ] generations_planned = N == ads_found = N
[ ] Copy skeleton extracted and output copy matches paragraph count
[ ] No em dashes or hyphens in copy output
[ ] Higgsfield Soul / Realistic confirmed as model
[ ] Prompt describes a realistic phone-camera scene (no studio, no AI-looking elements)
```

---

### BLOCK 4 — Generate in Higgsfield

```
Model: soul_2
Style: realistic
Count: 3
Prompt: [from Block 3]
Aspect ratio: [match reference]
```

**After 3 variations complete — select the most native:**

View all 3. Reject any that have:
- Extra or distorted limbs
- Unnatural facial expressions
- Obvious AI artifacts (perfect skin, geometric lighting)
- Weird angles no human would take
- Overly polished or editorial feel
- Any element that makes it look "generated"

Print selection:
```
SELECTION [nativa N]:
  Variation 1: [brief description] → [REJECT / CONSIDER]
  Variation 2: [brief description] → [REJECT / CONSIDER]
  Variation 3: [brief description] → [SELECTED — reason: most candid/natural]
```

If all 3 are rejected → regenerate once with a simpler, more grounded prompt.
If second attempt also fails → note it and move to next ad.

---

### BLOCK 5 — Save to desktop

```bash
mkdir -p "OUTPUT_FOLDER"
curl -L "SELECTED_HIGGSFIELD_URL" -o "OUTPUT_FOLDER/nativa_01.png"
```

Write `copy_nativa_0N.txt` — see `references/copy-format.md`.

Final output:
```
OUTPUT_FOLDER/
  nativa_01.png
  copy_nativa_01.txt
  nativa_02.png
  copy_nativa_02.txt
```

Print OUTPUT_FOLDER path. Done.

---

## Error handling

| Error | Action |
|-------|--------|
| All 3 variations look AI-generated | Simplify prompt (less detail, more candid description), regenerate once |
| Copy structure doesn't match skeleton | Recount paragraphs, fix before delivering |
| Meta Library blocked | Chrome MCP only — no Gemini, no web_fetch |
| AD_SOURCE unreadable | Ask user to re-export or provide differently |
