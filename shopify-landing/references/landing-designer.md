# Landing Page Designer — Build Reference

Loaded during Step 6 of the shopify-landing skill.
Defines the exact methodology for building and uploading the landing page.

All file operations use the REST API from the command line.
Do NOT use the Shopify browser admin — it does not render correctly in automated environments.

---

## Core rules (never break these)

1. **Hero = native section only.** Never create a custom `.liquid` file for the hero. Always use the theme's `product-information` section.
2. **One section = one file.** Every custom section is its own `.liquid` file. Never combine multiple sections in a single file.
3. **No third-party review apps.** Social proof sections are built custom with hardcoded reviews and generated reviewer photos.
4. **No image placeholders in the final page.** Every `<img>` tag must have a real source. If product photos don't exist yet, use neutral solid-color blocks styled in the brand palette — never grey boxes.
5. **Visual replication.** Colors, typography, layout density, and section order must match the competitor reference.

---

## Phase 1 — Build assets

### Step A — Analyze competitor structure

From the competitor analysis, you have `SECTION_STRUCTURE` — the ordered list of sections.
Remove the hero from this list (it's handled by the native section).
The remaining sections are the ones you'll build as custom `.liquid` files.

Example output:
```
Custom sections to build (in order):
1. benefits-grid
2. how-it-works
3. ingredients-overview
4. social-proof
5. faq
6. trust-bar
```

---

### Step B — Extract visual style from competitor

Before writing any Liquid, establish the design system from `VISUAL_STYLE`:

```
COLOR_PRIMARY:     [hex — main brand color, used for CTAs and accents]
COLOR_SECONDARY:   [hex — background or section fills]
COLOR_TEXT:        [hex — body text color]
COLOR_HEADING:     [hex — heading color, may differ from body]
FONT_HEADING:      [font name or stack — e.g. "Georgia, serif"]
FONT_BODY:         [font name or stack — e.g. "Inter, sans-serif"]
LAYOUT_WIDTH:      [max-width — e.g. 1200px]
SECTION_PADDING:   [vertical padding — e.g. 60px top/bottom]
```

These values are used as CSS custom properties in every section file.

---

### Step C — Write each `.liquid` section file

For each custom section, write a complete, self-contained `.liquid` file.

**File naming convention:** `section-[descriptive-name].liquid`

**CRITICAL — Every section MUST be fully editable from the Shopify theme editor.**
This means every piece of content (text, colors, images, labels) must be a schema setting — never hardcoded in the HTML. Follow this structure exactly:

```liquid
{% comment %} Section: [Section Name] {% endcomment %}

<style>
  .section-[name] {
    background: {{ section.settings.background_color }};
    padding: {{ section.settings.padding_top }}px 0 {{ section.settings.padding_bottom }}px;
    font-family: FONT_BODY;
    color: COLOR_TEXT;
  }
  .section-[name] .container {
    max-width: LAYOUT_WIDTH;
    margin: 0 auto;
    padding: 0 20px;
  }
  .section-[name] h2 {
    font-family: FONT_HEADING;
    color: {{ section.settings.heading_color }};
  }
  /* All other CSS here — use section.settings values for any user-editable property */
</style>

<div class="section-[name]">
  <div class="container">
    {% if section.settings.heading != blank %}
      <h2>{{ section.settings.heading }}</h2>
    {% endif %}
    {% if section.settings.subheading != blank %}
      <p class="subheading">{{ section.settings.subheading }}</p>
    {% endif %}

    {% for block in section.blocks %}
      {% case block.type %}
        {% when 'item' %}
          <div class="item" {{ block.shopify_attributes }}>
            {% if block.settings.title != blank %}
              <h3>{{ block.settings.title }}</h3>
            {% endif %}
            {% if block.settings.text != blank %}
              <div>{{ block.settings.text }}</div>
            {% endif %}
          </div>
      {% endcase %}
    {% endfor %}
  </div>
</div>

{% schema %}
{
  "name": "[Section Display Name]",
  "tag": "section",
  "class": "section-[name]",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "Heading",
      "default": "[Default heading text in OUTPUT_LANGUAGE]"
    },
    {
      "type": "textarea",
      "id": "subheading",
      "label": "Subheading",
      "default": "[Default subheading in OUTPUT_LANGUAGE]"
    },
    {
      "type": "color",
      "id": "background_color",
      "label": "Background color",
      "default": "COLOR_SECONDARY"
    },
    {
      "type": "color",
      "id": "heading_color",
      "label": "Heading color",
      "default": "COLOR_TEXT"
    },
    {
      "type": "range",
      "id": "padding_top",
      "label": "Padding top",
      "min": 0,
      "max": 120,
      "step": 4,
      "unit": "px",
      "default": 60
    },
    {
      "type": "range",
      "id": "padding_bottom",
      "label": "Padding bottom",
      "min": 0,
      "max": 120,
      "step": 4,
      "unit": "px",
      "default": 60
    }
  ],
  "blocks": [
    {
      "type": "item",
      "name": "Item",
      "settings": [
        {
          "type": "text",
          "id": "title",
          "label": "Title",
          "default": "[Item title in OUTPUT_LANGUAGE]"
        },
        {
          "type": "richtext",
          "id": "text",
          "label": "Text",
          "default": "<p>[Item text in OUTPUT_LANGUAGE]</p>"
        }
      ]
    }
  ],
  "presets": [
    {
      "name": "[Section Display Name]",
      "blocks": [
        { "type": "item" },
        { "type": "item" },
        { "type": "item" }
      ]
    }
  ]
}
{% endschema %}
```

**Rules for schema:**
- Every text string visible to the customer must be a schema setting with a `default` value in `OUTPUT_LANGUAGE`
- Every color must be a `color` type setting with a default from the brand palette
- Images must use `"type": "image_picker"` settings — never hardcoded `src` URLs
- Blocks are used for repeating elements (benefit items, FAQ entries, review cards, steps)
- `{{ block.shopify_attributes }}` must be on every block wrapper div — this enables drag-to-reorder in the editor
- The `presets` array must always be present — this makes the section appear in "Add section" in the editor

**Adapt the schema per section type:**

For image sections, add to settings:
```json
{ "type": "image_picker", "id": "image", "label": "Image" }
```
And in HTML: `{% if section.settings.image %}<img src="{{ section.settings.image | image_url: width: 1200 }}" alt="{{ section.settings.heading }}">{% endif %}`

For CTA buttons, add to settings:
```json
{ "type": "text", "id": "button_label", "label": "Button label", "default": "[CTA text in OUTPUT_LANGUAGE]" },
{ "type": "url", "id": "button_url", "label": "Button URL" }
```

For before/after sections, use two `image_picker` settings: `image_before` and `image_after`.

> ⚠️ Never hardcode any customer-visible text, color, or image URL directly in the HTML.
> Everything the user might want to change must be a schema setting.
> If a section is not editable from the theme editor, it is wrong — rewrite it.

**Copy rules — apply COPY_STRATEGY first:**

Before writing a single word of copy, check `COPY_STRATEGY`:

**A — Copy 1:1:** Use competitor copy verbatim. If `COPY_LANGUAGE` differs, translate maintaining exact meaning, tone, and structure. Do not improve or rewrite.

**B — Translate:** Translate all copy into `COPY_LANGUAGE`. Preserve sentence structure, emotional triggers, power words (find equivalents), CTA phrasing, and urgency signals. Do not summarize.

**C — Rewrite:** Keep section order, core angles, benefit claims, and objection-handling structure. Rewrite every sentence from scratch — no phrase over 4 words can match the original. Push the tone: if clinical, make it more authoritative; if bold, bolder.

**D — New copy from research (use when competitor sells a DIFFERENT product):**
Before writing anything, read `PRODUCT_DOC` completely. Extract and store `product_research`:
- Avatar pain points — ideally in the exact words they use
- Avatar desire / transformation they want
- Product benefits (top 5-7, ranked by emotional impact)
- Differentiators vs alternatives
- Tone of voice
- FAQs and objections
- Guarantee / risk reversal

From the competitor PDF, extract ONLY:
- Section order and types (problem section, solution, ingredients, timeline, comparison, etc.)
- Narrative flow (how they move from hook → problem → solution → proof → CTA)
- Visual structure per section (layout, image position, card style)
- Colors and fonts (if VISUAL_MODE = REPLICATE)

Writing rules for Mode D:
- Every headline, bullet, claim → sourced from `product_research` only
- Never use competitor's product claims, ingredients, or specific results
- Use competitor's ARGUMENT STRUCTURE (e.g. "problem → why traditional solutions fail → science-backed solution") but populate it with YOUR product's story
- If competitor has a "5 ingredients" section → build an equivalent section with YOUR ingredients
- If competitor has a "week by week results" timeline → build equivalent with YOUR product's timeline
- Zero claims from the competitor's product — only the architecture

**Visual rules — apply VISUAL_MODE:**

**VISUAL_MODE = REPLICATE:** Use colors and fonts extracted from competitor analysis exactly. The page must feel like the same brand family.

**VISUAL_MODE = CUSTOM:** Use user-confirmed `COLOR_PRIMARY`, `COLOR_SECONDARY`, `COLOR_TEXT`, `COLOR_ACCENT`. Select font pairing from these by product category:
- Health/beauty/skincare → `'Playfair Display', serif` + `'Inter', sans-serif`
- Supplements/clinical → `'DM Serif Display', serif` + `'DM Sans', sans-serif`
- Home/utility/pest control → `'Montserrat', sans-serif` + `'Open Sans', sans-serif`
- Premium/luxury → `'Cormorant Garamond', serif` + `'Jost', sans-serif`
- Bold/direct response → `'Oswald', sans-serif` + `'Source Sans Pro', sans-serif`

Load via Google Fonts `<link>` tag inside each section's `<style>` block.

**CSS quality rules (always apply regardless of mode):**
- Use CSS Grid and Flexbox — no float layouts, no table-based structure
- Alternate section backgrounds: light sections (#FAFAFA or COLOR_SECONDARY) and dark sections (COLOR_DARK or near-black) to create visual rhythm
- Cards must have: `border-radius: 8–12px`, subtle `box-shadow`, and `padding: 24–32px`
- All CTAs: `border-radius: 4–6px`, `padding: 16px 32px`, `font-weight: 700`, `letter-spacing: 0.5px`
- Mobile-first: every section must be responsive with `@media (max-width: 768px)` breakpoints
- Sections must have generous padding: minimum `80px` top/bottom on desktop, `48px` on mobile

---

### Step D — Upload all section files to the theme

For each `.liquid` file:

```bash
curl -X PUT "https://STORE_URL/admin/api/2024-01/themes/WORKING_THEME_ID/assets.json" \
  -H "X-Shopify-Access-Token: ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "asset": {
      "key": "sections/section-[name].liquid",
      "value": "[full file content escaped as JSON string]"
    }
  }'
```

Verify each upload returns HTTP 200 and the asset key matches.
Log all successful uploads before moving to the template step.

---

### Step E — Build the template JSON

Assemble the template in `SECTION_STRUCTURE` order:

```json
{
  "sections": {
    "main": {
      "type": "main-product",
      "disabled": false,
      "settings": {
        "show_vendor": false,
        "show_rating": false
      }
    },
    "benefits": {
      "type": "section-benefits-grid",
      "disabled": false,
      "settings": {}
    },
    "how-it-works": {
      "type": "section-how-it-works",
      "disabled": false,
      "settings": {}
    },
    "social-proof": {
      "type": "section-social-proof",
      "disabled": false,
      "settings": {}
    },
    "faq": {
      "type": "section-faq",
      "disabled": false,
      "settings": {}
    }
  },
  "order": ["main", "benefits", "how-it-works", "social-proof", "faq"]
}
```

Rules:
- `main` (native `main-product`) is always first.
- Section type names must exactly match the uploaded `.liquid` filenames without the `.liquid` extension.
- The `order` array controls render order on the page.

---

### Step F — Upload the template JSON

```bash
curl -X PUT "https://STORE_URL/admin/api/2024-01/themes/WORKING_THEME_ID/assets.json" \
  -H "X-Shopify-Access-Token: ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "asset": {
      "key": "templates/product.TEMPLATE_NAME.json",
      "value": "[template JSON escaped as string]"
    }
  }'
```

Verify the upload returns the correct asset key.

---

## Phase 2 — Configure and QA

### Step G — Assign template to product

```graphql
mutation {
  productUpdate(input: {
    id: "PRODUCT_ID"
    templateSuffix: "TEMPLATE_NAME"
  }) {
    product { id templateSuffix }
    userErrors { field message }
  }
}
```

If `write_products` scope missing → report to user:
> "Manual action required: go to Shopify Admin > Products > [PRODUCT_TITLE] > scroll to 'Theme template' > select `product.TEMPLATE_NAME` > Save."

### Step H — QA via Claude in Chrome

Navigate to: `https://STORE_URL/products/PRODUCT_HANDLE`

Check each item:
- [ ] All custom sections are visible on the page
- [ ] Hero (Add to Cart, price, variant picker) works
- [ ] No grey placeholder boxes — all image slots handled
- [ ] Social proof section shows custom reviews (no third-party widget iframes)
- [ ] Colors match `COLOR_PRIMARY`, `COLOR_SECONDARY`, `COLOR_TEXT`
- [ ] Typography matches `FONT_HEADING`, `FONT_BODY`
- [ ] Section order matches `SECTION_STRUCTURE`

For any failed check: patch the specific `.liquid` file via API and re-verify.

---

## Phase 3 — Pixel-match and handoff

### Step I — Side-by-side comparison

1. Screenshot the competitor URL.
2. Screenshot the live product page.
3. Present both to the user for review.

### Step J — Handoff report

```
=== LANDING PAGE COMPLETE ===

Live URL:   https://STORE_URL/products/PRODUCT_HANDLE
Template:   product.TEMPLATE_NAME

Files created:
  sections/section-benefits-grid.liquid
  sections/section-how-it-works.liquid
  sections/section-[...].liquid
  templates/product.TEMPLATE_NAME.json

To replace product photos later:
  Online Store > Themes > Customize > navigate to product page
  > click each image block > upload your photo

Manual actions remaining:
  [ List any steps that required manual intervention ]

Next: Higgsfield photo generation phase
==============================
```
