# Competitor Analysis — Extraction Guide

This file is loaded during Step 1 of the shopify-landing skill.
It defines exactly what to extract from a competitor's product page (PDF or URL).

---

## Input handling — PDF only

**The PDF is the single source of truth.** Do not fetch the URL. Do not open Chrome. Do not visit the competitor page.

Read the PDF thoroughly and extract everything from it:
1. Section structure top to bottom — number each section
2. All copy, headings, subheadings, bullets
3. Colors visible (backgrounds, buttons, text, accents)
4. Fonts visible (serif/sans, weight, style)
5. Buy box elements in order
6. Media type per section (photo/video/illustration — only call it video if ▶ is visible)
7. Announcement bar content and style
8. Header style
9. Footer structure and colors
---

## Extraction checklist

Work through each item. Mark as `N/A` if genuinely not present.

### Product basics
- **Product name** — exact name used
- **Product category** — what Shopify category this maps to (e.g. Health & Beauty > Vitamins, Sports > Equipment)
- **Primary use case** — one sentence: what does this product do?
- **Format / variants** — sizes, colors, flavors, bundles

### Positioning
- **Target audience** — who is this for? (age, lifestyle, pain point)
- **Main problem it solves** — the core customer pain
- **Key benefits** — list every claim made (3–8 typically)
- **Differentiators** — what makes it different from alternatives?
- **Social proof signals** — reviews, ratings, certifications, awards, press mentions

### Pricing
- **Listed price** — exact price shown
- **Compare-at / original price** — if shown (crossed out)
- **Pricing framing** — "per serving", "per day", subscription discount, etc.

### Copy & tone
- **Headline** — exact headline text
- **Subheadline** — if present
- **Tone** — choose the closest: Premium / Clinical / Playful / Bold / Minimal / Warm / Urgent
- **Writing style** — short punchy bullets? Long-form storytelling? FAQ-heavy?
- **Power words used** — list any repeated emotional or conversion words

### Page structure
Document every section in order from top to bottom:

```
1. [Section type] — [brief description]
   e.g. "Hero — full-width image, headline, CTA button"
2. [Section type] — [brief description]
   e.g. "Benefits grid — 4 icons with text"
3. ...
```

Common section types to identify:
- Hero (headline + image + CTA)
- Benefits / features grid
- How it works / process steps
- Ingredients / materials / specs
- Social proof / reviews
- Comparison table
- FAQ accordion
- Trust badges / guarantees
- Before/after
- Founder story / brand story
- Urgency / scarcity block
- Final CTA / sticky add-to-cart

### Visual style
- **Color palette** — primary, secondary, accent colors (describe or hex if visible)
- **Image style** — lifestyle photography / studio white / illustration / UGC
- **Layout density** — minimal whitespace / packed with content / balanced
- **Typography feel** — serif / sans-serif, large headings / small, editorial / functional

---

## Name generation rules

Generate 3 product name options using this framework:

**Option A — Descriptive** (what it literally is or does)
- Format: `[Adjective] + [Product Type]` or `[Key Ingredient] + [Form]`
- Example: "Advanced Collagen Complex", "Cold-Pressed Vitamin C Serum"
- Rule: clear, searchable, no ambiguity

**Option B — Benefit-led** (the transformation the customer gets)
- Format: `[Outcome] + [Product Type]` or verb-led
- Example: "Glow Daily Serum", "Deep Sleep Formula", "Lift & Define Cream"
- Rule: emotionally resonant, promise-focused

**Option C — Brand-style** (short, memorable, ownable)
- Format: 1–2 words max, could be a coined word or unexpected pairing
- Example: "Lumify", "Arctic Boost", "The One Serum"
- Rule: distinctive, avoids generic descriptors

**Naming rules to always follow:**
- No trademark violations — avoid competitor brand names
- No unsubstantiated superlatives ("world's best", "#1")
- Keep under 60 characters for SEO
- Avoid special characters except hyphens
- If it's a supplement or health product: no medical claims in the name

---

## Media type detection (mandatory for every section)

Before building any section, identify the exact media type the competitor uses:

```
For each section, extract:
  media_type:     photo | video | illustration | diagram | gif | none
  media_layout:   carousel_horizontal | carousel_vertical | grid | static | slider | single
  media_count:    how many items? (e.g. "8 photos in horizontal scroll")
  has_text_overlay: yes/no
  media_position: left | right | top | bottom | background | none
```

**Common mistakes to avoid:**
- Competitor shows photos with play button overlay → those ARE videos (video testimonials)
- Competitor shows photos scrolling left/right → photo carousel, NOT videos
- Competitor shows 3 customer faces in static grid → static photo grid, NOT carousel
- Competitor shows product surrounded by ingredients → product flat lay photo, NOT lifestyle video

**Rule:** If you cannot clearly see a play button (▶) in the competitor's image, it is a PHOTO, not a video.

## Icon style detection

Before building the above-the-fold or any section, identify whether the competitor uses emojis or SVG icons:

**Check these elements in the competitor PDF:**
- Trust badges (🔄 vs SVG icon)
- Benefit bullets (✓ emoji vs SVG checkmark)
- Announcement bar items (| dividers vs icon+text)
- Shipping/guarantee badges

**If competitor uses SVG icons** (rendered icons, not emoji characters):
- Generate equivalent inline SVG for each icon
- Common patterns:
  - Shield: `<svg viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>`
  - Truck/shipping: `<svg viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2"><rect x="1" y="3" width="15" height="13"/><polygon points="16 8 20 8 23 11 23 16 16 16 16 8"/><circle cx="5.5" cy="18.5" r="2.5"/><circle cx="18.5" cy="18.5" r="2.5"/></svg>`
  - Checkmark circle: `<svg viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><polyline points="22 4 12 14.01 9 11.01"/></svg>`
  - Return/arrows: `<svg viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2"><polyline points="1 4 1 10 7 10"/><path d="M3.51 15a9 9 0 1 0 .49-3.5"/></svg>`
  - Leaf/natural: `<svg viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2"><path d="M2 22c0-6.08 3.66-11.26 9-13.4A13 13 0 0 1 22 22H2z"/><path d="M12 8.5V22"/></svg>`

**If competitor uses emojis** → use the same emojis.

Store the detected icon style as `ICON_STYLE: SVG` or `ICON_STYLE: EMOJI` and apply consistently across all sections.
