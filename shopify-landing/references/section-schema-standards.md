# Section Schema Standards — Reference

Loaded during Step 6 of the shopify-landing skill.
Defines the minimum schema quality standard for every custom section Claude creates.
Inspired by Elixir theme's section schemas — the benchmark for editability.

---

## Core principle

Every section Claude creates must be fully editable from the Shopify theme customizer.
A merchant should never need to touch code to adjust colors, sizes, spacing, or copy.
This is the same standard Elixir achieves with every section.

---

## Mandatory schema groups (every section must include all of these)

### 1. Theme integration
```json
{
  "type": "checkbox",
  "id": "use_theme_colors",
  "label": "Use Theme Colors",
  "default": true,
  "info": "When enabled, uses global theme colors. When disabled, use custom colors below."
}
```

### 2. Section padding (always 4 settings with px unit)
```json
{ "type": "header", "content": "Section Padding" },
{ "type": "range", "id": "padding_top",    "min": 0, "max": 100, "step": 5, "unit": "px", "label": "Top Padding",    "default": 50 },
{ "type": "range", "id": "padding_bottom", "min": 0, "max": 100, "step": 5, "unit": "px", "label": "Bottom Padding", "default": 50 },
{ "type": "range", "id": "padding_left",   "min": 0, "max": 100, "step": 5, "unit": "px", "label": "Left Padding",   "default": 20 },
{ "type": "range", "id": "padding_right",  "min": 0, "max": 100, "step": 5, "unit": "px", "label": "Right Padding",  "default": 20 }
```

### 3. Section max width
```json
{ "type": "range", "id": "section_max_width", "min": 600, "max": 1600, "step": 50, "unit": "px", "label": "Section Max Width", "default": 1200, "info": "Maximum width of the content container" }
```

### 4. Section background (always full background control)
```json
{ "type": "header", "content": "Section Background" },
{ "type": "select", "id": "background_type", "label": "Background Type",
  "options": [
    { "value": "solid",    "label": "Solid Color" },
    { "value": "gradient", "label": "Gradient" },
    { "value": "image",    "label": "Background Image" }
  ], "default": "solid" },
{ "type": "color",            "id": "section_bg",          "label": "Background Color",          "default": "#ffffff",                                      "info": "Used when Solid Color is selected" },
{ "type": "color_background", "id": "section_bg_gradient",  "label": "Background Gradient",        "default": "linear-gradient(135deg, #ffffff, #f8f8f8)",    "info": "Used when Gradient is selected" },
{ "type": "image_picker",     "id": "bg_image_desktop",     "label": "Background Image (Desktop)", "info": "Used when Background Image is selected" },
{ "type": "image_picker",     "id": "bg_image_mobile",      "label": "Background Image (Mobile)",  "info": "Optional: different image for mobile. Falls back to desktop if not set" },
{ "type": "select", "id": "bg_image_size",     "label": "Image Size",     "options": [{"value":"cover","label":"Cover"},{"value":"contain","label":"Contain"},{"value":"auto","label":"Auto"}], "default": "cover" },
{ "type": "select", "id": "bg_image_position", "label": "Image Position", "options": [{"value":"center center","label":"Center"},{"value":"top center","label":"Top"},{"value":"bottom center","label":"Bottom"}], "default": "center center" },
{ "type": "checkbox", "id": "bg_overlay_enable",  "label": "Enable Overlay",      "default": false },
{ "type": "color",    "id": "bg_overlay_color",   "label": "Overlay Color",        "default": "#000000" },
{ "type": "range",    "id": "bg_overlay_opacity", "label": "Overlay Opacity", "min": 0, "max": 1, "step": 0.05, "default": 0.3 }
```

---

## Typography settings pattern (desktop + mobile always separate)

For every heading in a section:
```json
{ "type": "header", "content": "Heading Typography" },
{ "type": "text",   "id": "heading_text",           "label": "Heading Text" },
{ "type": "text",   "id": "heading_accent_text",    "label": "Accent Text (highlighted part)" },
{ "type": "color",  "id": "heading_color",          "label": "Heading Color",       "default": "#111111" },
{ "type": "color",  "id": "heading_accent_color",   "label": "Accent Color",        "default": "#1a4731" },
{ "type": "range",  "id": "heading_size_desktop",   "label": "Heading Size Desktop","min": 24, "max": 72, "step": 1, "unit": "px", "default": 40 },
{ "type": "range",  "id": "heading_size_mobile",    "label": "Heading Size Mobile", "min": 20, "max": 56, "step": 1, "unit": "px", "default": 28 },
{ "type": "select", "id": "heading_weight",         "label": "Font Weight",
  "options": [{"value":"400","label":"Regular"},{"value":"500","label":"Medium"},{"value":"600","label":"Semibold"},{"value":"700","label":"Bold"},{"value":"800","label":"Extrabold"}],
  "default": "700" },
{ "type": "range",  "id": "heading_letter_spacing", "label": "Letter Spacing", "min": -2, "max": 10, "step": 0.5, "unit": "px", "default": 0 },
{ "type": "range",  "id": "heading_margin_bottom",  "label": "Heading Margin Bottom", "min": 0, "max": 60, "step": 2, "unit": "px", "default": 16 }
```

For subheadings:
```json
{ "type": "textarea", "id": "subheading",           "label": "Subheading" },
{ "type": "color",    "id": "subheading_color",     "label": "Subheading Color",       "default": "#666666" },
{ "type": "range",    "id": "subheading_size_desktop","label": "Subheading Size Desktop","min": 12, "max": 28, "step": 1, "unit": "px", "default": 16 },
{ "type": "range",    "id": "subheading_size_mobile", "label": "Subheading Size Mobile", "min": 12, "max": 22, "step": 1, "unit": "px", "default": 14 },
{ "type": "select",   "id": "subheading_weight",    "label": "Subheading Weight",
  "options": [{"value":"300","label":"Light"},{"value":"400","label":"Regular"},{"value":"500","label":"Medium"},{"value":"600","label":"Semibold"},{"value":"700","label":"Bold"}],
  "default": "400" },
{ "type": "range",    "id": "subheading_margin_bottom","label": "Subheading Margin Bottom","min": 0, "max": 60, "step": 2, "unit": "px", "default": 24 }
```

For body/paragraph text:
```json
{ "type": "color",  "id": "body_color",        "label": "Body Text Color", "default": "#555555" },
{ "type": "range",  "id": "body_size_desktop", "label": "Body Size Desktop","min": 12, "max": 22, "step": 1, "unit": "px", "default": 16 },
{ "type": "range",  "id": "body_size_mobile",  "label": "Body Size Mobile", "min": 12, "max": 18, "step": 1, "unit": "px", "default": 14 },
{ "type": "range",  "id": "body_line_height",  "label": "Line Height", "min": 1, "max": 2.5, "step": 0.1, "default": 1.6 }
```

---

## Card/block settings pattern

For any section that displays cards or repeated blocks:
```json
{ "type": "header", "content": "Card Settings" },
{ "type": "color",  "id": "card_bg",           "label": "Card Background",   "default": "#ffffff" },
{ "type": "range",  "id": "card_border_radius", "label": "Card Border Radius","min": 0, "max": 40, "step": 2, "unit": "px", "default": 10 },
{ "type": "checkbox","id": "card_show_border",  "label": "Show Card Border",  "default": true },
{ "type": "range",  "id": "card_border_width",  "label": "Border Width",      "min": 0, "max": 6,  "step": 1, "unit": "px", "default": 1 },
{ "type": "select", "id": "card_border_style",  "label": "Border Style",
  "options": [{"value":"solid","label":"Solid"},{"value":"dashed","label":"Dashed"},{"value":"dotted","label":"Dotted"}],
  "default": "solid" },
{ "type": "color",  "id": "card_border_color",  "label": "Border Color",      "default": "#e2e8f0" },
{ "type": "checkbox","id": "card_show_shadow",  "label": "Show Card Shadow",  "default": false },
{ "type": "color",  "id": "card_shadow_color",  "label": "Shadow Color",      "default": "rgba(0,0,0,0.08)" },
{ "type": "range",  "id": "card_padding_v",     "label": "Card Padding Vertical",  "min": 8, "max": 40, "step": 2, "unit": "px", "default": 20 },
{ "type": "range",  "id": "card_padding_h",     "label": "Card Padding Horizontal","min": 8, "max": 40, "step": 2, "unit": "px", "default": 20 },
{ "type": "range",  "id": "card_gap",           "label": "Gap Between Cards",      "min": 8, "max": 48, "step": 4, "unit": "px", "default": 20 }
```

---

## CTA button settings pattern

For sections with a CTA button:
```json
{ "type": "header",  "content": "CTA Button" },
{ "type": "text",    "id": "cta_text",          "label": "Button Text",           "default": "Comprar ahora" },
{ "type": "url",     "id": "cta_url",           "label": "Button URL" },
{ "type": "color",   "id": "cta_bg",            "label": "Button Background",     "default": "#111111" },
{ "type": "color",   "id": "cta_text_color",    "label": "Button Text Color",     "default": "#ffffff" },
{ "type": "color",   "id": "cta_border_color",  "label": "Button Border Color",   "default": "#111111" },
{ "type": "range",   "id": "cta_border_width",  "label": "Border Width",          "min": 0, "max": 4, "step": 1, "default": 0 },
{ "type": "range",   "id": "cta_border_radius", "label": "Border Radius",         "min": 0, "max": 50, "step": 2, "unit": "px", "default": 8 },
{ "type": "range",   "id": "cta_font_size",     "label": "Font Size",             "min": 12, "max": 22, "step": 1, "unit": "px", "default": 16 },
{ "type": "select",  "id": "cta_font_weight",   "label": "Font Weight",
  "options": [{"value":"400","label":"Regular"},{"value":"600","label":"Semibold"},{"value":"700","label":"Bold"}], "default": "700" },
{ "type": "range",   "id": "cta_padding_v",     "label": "Padding Vertical",      "min": 8, "max": 28, "step": 2, "unit": "px", "default": 14 },
{ "type": "range",   "id": "cta_padding_h",     "label": "Padding Horizontal",    "min": 12, "max": 60, "step": 4, "unit": "px", "default": 28 },
{ "type": "select",  "id": "cta_text_transform","label": "Text Transform",
  "options": [{"value":"none","label":"None"},{"value":"uppercase","label":"Uppercase"},{"value":"capitalize","label":"Capitalize"}], "default": "none" },
{ "type": "range",   "id": "cta_letter_spacing", "label": "Letter Spacing",       "min": 0, "max": 4, "step": 0.5, "unit": "px", "default": 0 },
{ "type": "checkbox","id": "cta_full_width",     "label": "Full Width Button",     "default": false }
```

---

## Layout/columns pattern (for grid sections)

```json
{ "type": "header", "content": "Layout" },
{ "type": "select", "id": "columns_desktop",
  "label": "Columns (Desktop)",
  "options": [{"value":"2","label":"2"},{"value":"3","label":"3"},{"value":"4","label":"4"}],
  "default": "3" },
{ "type": "select", "id": "columns_mobile",
  "label": "Columns (Mobile)",
  "options": [{"value":"1","label":"1"},{"value":"2","label":"2"}],
  "default": "1",
  "info": "Number of columns on mobile devices" },
{ "type": "select", "id": "text_align",
  "label": "Text Alignment",
  "options": [{"value":"left","label":"Left"},{"value":"center","label":"Center"},{"value":"right","label":"Right"}],
  "default": "center" },
{ "type": "select", "id": "image_position",
  "label": "Media Position (Desktop)",
  "options": [{"value":"left","label":"Left of content"},{"value":"right","label":"Right of content"}],
  "default": "right",
  "info": "Only applies to two-column layouts" }
```

---

## Divider/separator pattern

```json
{ "type": "header",   "content": "Divider" },
{ "type": "checkbox", "id": "show_divider",       "label": "Show Section Divider",  "default": false },
{ "type": "color",    "id": "divider_color",       "label": "Divider Color",         "default": "#e5e7eb" },
{ "type": "range",    "id": "divider_width",       "label": "Divider Thickness",     "min": 1, "max": 4, "step": 1, "unit": "px", "default": 1 },
{ "type": "select",   "id": "divider_position",    "label": "Divider Position",      "options": [{"value":"top","label":"Top"},{"value":"bottom","label":"Bottom"},{"value":"both","label":"Both"}], "default": "bottom" }
```

---

## Icon settings pattern (for sections with SVG icons)

```json
{ "type": "header",   "content": "Icon Settings" },
{ "type": "select",   "id": "icon_style",
  "label": "Icon Style",
  "options": [
    {"value":"svg",    "label":"SVG Icon (auto-generated)"},
    {"value":"emoji",  "label":"Emoji"},
    {"value":"image",  "label":"Custom Image"},
    {"value":"none",   "label":"No Icon"}
  ], "default": "svg" },
{ "type": "range",    "id": "icon_size",          "label": "Icon Size",            "min": 16, "max": 80, "step": 2, "unit": "px", "default": 32 },
{ "type": "color",    "id": "icon_color",         "label": "Icon Color",           "default": "#111111" },
{ "type": "color",    "id": "icon_bg",            "label": "Icon Background",      "default": "transparent" },
{ "type": "range",    "id": "icon_bg_size",       "label": "Icon Container Size",  "min": 24, "max": 100, "step": 4, "unit": "px", "default": 48 },
{ "type": "range",    "id": "icon_border_radius", "label": "Icon Border Radius",   "min": 0,  "max": 50,  "step": 2, "unit": "px", "default": 8 },
{ "type": "range",    "id": "icon_margin_bottom", "label": "Icon Margin Bottom",   "min": 0,  "max": 30,  "step": 2, "unit": "px", "default": 12 }
```

---

## Scrolling ticker pattern (for announcement marquee bars)

```json
{ "type": "range",    "id": "scroll_speed",      "label": "Scroll Speed (lower = faster)", "min": 5, "max": 60, "step": 1, "unit": "s", "default": 20 },
{ "type": "checkbox", "id": "show_dividers",     "label": "Show dividers between items",   "default": true },
{ "type": "select",   "id": "divider_type",      "label": "Divider Type",
  "options": [{"value":"text","label":"Text character"},{"value":"dot","label":"Dot"},{"value":"pipe","label":"Pipe (|)"},{"value":"star","label":"Star (★)"}],
  "default": "dot" },
{ "type": "text",     "id": "divider_char",      "label": "Custom Divider Character", "default": "·" }
```

---

## Star rating display pattern

```json
{ "type": "header",   "content": "Star Rating" },
{ "type": "checkbox", "id": "show_rating",       "label": "Show Star Rating",       "default": true },
{ "type": "text",     "id": "rating_text",       "label": "Rating Text",            "default": "4.9 · Excellent · 4,537 reviews" },
{ "type": "color",    "id": "star_color",        "label": "Star Color",             "default": "#f59e0b" },
{ "type": "range",    "id": "star_size",         "label": "Star Size",              "min": 12, "max": 28, "step": 1, "unit": "px", "default": 18 },
{ "type": "color",    "id": "rating_text_color", "label": "Rating Text Color",      "default": "#374151" },
{ "type": "range",    "id": "rating_font_size",  "label": "Rating Font Size",       "min": 11, "max": 16, "step": 1, "unit": "px", "default": 13 },
{ "type": "select",   "id": "rating_layout",     "label": "Rating Layout",
  "options": [{"value":"inline","label":"Inline (stars + text)"},{"value":"stacked","label":"Stacked"}],
  "default": "inline" }
```

---

## Section-level application rules

### Every section Claude creates MUST have:
1. `use_theme_colors` checkbox
2. Full padding group (top, bottom, left, right)
3. `section_max_width` range
4. Background type selector (solid/gradient/image) with all sub-settings
5. Desktop AND mobile font sizes for every text element (never just one)

### Sections with headings MUST have:
- Two-part heading (main text + accent text) with separate colors
- Heading size for desktop AND mobile
- Font weight selector
- Letter spacing
- Margin bottom

### Sections with repeated items (reviews, benefits, FAQs, steps) MUST use blocks:
- Each item is a block, not hardcoded
- Minimum 1 block per content type
- Block limit appropriate to the section (reviews: 10, benefits: 8, steps: 6, FAQ: 12)

### Sections with buttons MUST have:
- Full button settings group (bg, text color, border, radius, size, weight, padding, transform)

### Sections with images MUST have:
- `image_position` for desktop (left/right)
- Separate mobile behavior option
- Border radius for images
- Aspect ratio option where relevant

---

## Scrolling ticker implementation

When the competitor has a scrolling marquee announcement bar, create `sections/scrolling-ticker.liquid` matching Elixir's pattern:

```liquid
<div class="scrolling-ticker" style="background: {{ section.settings.background_color }}; padding: 10px 0; overflow: hidden;">
  <div class="ticker-track" style="
    display: flex;
    white-space: nowrap;
    animation: ticker-scroll {{ section.settings.scroll_speed }}s linear infinite;
  ">
    {%- assign items_repeated = '' -%}
    {%- for i in (1..3) -%}
      {%- for block in section.blocks -%}
        {%- if block.type == 'announcement' -%}
        <span class="ticker-item" style="
          color: {{ section.settings.text_color }};
          font-weight: {{ section.settings.font_weight }};
          font-size: {{ section.settings.font_size }}px;
          padding: 0 24px;
        " {{ block.shopify_attributes }}>{{ block.settings.text }}</span>
        {%- if section.settings.show_dividers -%}
        <span style="color: {{ section.settings.text_color }}; opacity: 0.5; padding: 0 4px;">
          {%- case section.settings.divider_type -%}
            {%- when 'pipe' -%}|
            {%- when 'dot' -%}·
            {%- when 'star' -%}★
            {%- else -%}{{ section.settings.divider_char }}
          {%- endcase -%}
        </span>
        {%- endif -%}
        {%- endif -%}
      {%- endfor -%}
    {%- endfor -%}
  </div>
</div>

<style>
@keyframes ticker-scroll {
  0%   { transform: translateX(0); }
  100% { transform: translateX(-33.333%); }
}
.scrolling-ticker { width: 100%; }
.ticker-track { width: max-content; }
</style>

{% schema %}
{
  "name": "Scrolling Ticker",
  "settings": [
    { "type": "color",  "id": "background_color", "label": "Background Color", "default": "#000000" },
    { "type": "color",  "id": "text_color",        "label": "Text Color",       "default": "#ffffff" },
    { "type": "range",  "id": "font_size",         "label": "Font Size", "min": 10, "max": 20, "step": 1, "unit": "px", "default": 13 },
    { "type": "select", "id": "font_weight",       "label": "Font Weight",
      "options": [{"value":"400","label":"Regular"},{"value":"500","label":"Medium"},{"value":"600","label":"Semibold"},{"value":"700","label":"Bold"}],
      "default": "600" },
    { "type": "range",  "id": "scroll_speed",      "label": "Scroll Speed (lower=faster)", "min": 5, "max": 60, "step": 1, "unit": "s", "default": 20 },
    { "type": "header", "content": "Dividers" },
    { "type": "checkbox","id": "show_dividers",    "label": "Show Dividers",    "default": true },
    { "type": "select", "id": "divider_type",      "label": "Divider Type",
      "options": [{"value":"dot","label":"Dot (·)"},{"value":"pipe","label":"Pipe (|)"},{"value":"star","label":"Star (★)"},{"value":"custom","label":"Custom"}],
      "default": "dot" },
    { "type": "text",   "id": "divider_char",      "label": "Custom Divider Character", "default": "·" }
  ],
  "blocks": [
    {
      "type": "announcement",
      "name": "Ticker Item",
      "settings": [
        { "type": "text", "id": "text", "label": "Text", "default": "FREE SHIPPING" }
      ]
    }
  ],
  "presets": [{ "name": "Scrolling Ticker" }]
}
{% endschema %}
```

---

## Mobile-first CSS pattern (mandatory for every section)

Always write CSS mobile-first. Desktop is the override, not the base.

```css
/* ─── MOBILE BASE (default, no media query) ─── */
.section-name {
  padding: 28px 20px;
  width: 100%;
  box-sizing: border-box;
}
.section-name__heading {
  font-size: 22px;    /* mobile heading */
  line-height: 1.2;
  margin-bottom: 12px;
}
.section-name__body {
  font-size: 14px;
  line-height: 1.6;
}
.section-name__grid {
  display: flex;
  flex-direction: column;  /* stack on mobile */
  gap: 16px;
}
.section-name__cta {
  width: 100%;             /* full width on mobile */
  min-height: 52px;        /* minimum touch target */
  font-size: 16px;
}
.section-name__image {
  width: 100%;
  height: auto;
}

/* ─── DESKTOP OVERRIDE ─── */
@media (min-width: 750px) {
  .section-name {
    padding: 50px 40px;
  }
  .section-name__heading {
    font-size: 36px;
    margin-bottom: 20px;
  }
  .section-name__body {
    font-size: 16px;
  }
  .section-name__grid {
    flex-direction: row;   /* side by side on desktop */
    gap: 32px;
  }
  .section-name__cta {
    width: auto;           /* auto width on desktop */
    padding: 16px 32px;
  }
}
```

## Mobile schema settings pattern

Every section must include mobile-specific padding and font size settings:

```json
{ "type": "header", "content": "Mobile Padding" },
{ "type": "range", "id": "padding_top_mobile",    "label": "Top padding (mobile)",    "min": 0, "max": 60, "step": 4, "unit": "px", "default": 28 },
{ "type": "range", "id": "padding_bottom_mobile", "label": "Bottom padding (mobile)",  "min": 0, "max": 60, "step": 4, "unit": "px", "default": 28 },
{ "type": "range", "id": "padding_h_mobile",      "label": "Horizontal padding (mobile)","min": 0, "max": 40, "step": 4, "unit": "px", "default": 20 }
```

And in the Liquid, apply them:
```liquid
<style>
  @media (max-width: 749px) {
    .section-{{ section.id }} {
      padding: {{ section.settings.padding_top_mobile }}px {{ section.settings.padding_h_mobile }}px {{ section.settings.padding_bottom_mobile }}px !important;
    }
  }
</style>
```

## Quality checklist — before uploading any section

Before writing a section to the store, verify ALL of these:

**Structure:**
- [ ] `use_theme_colors` toggle present
- [ ] `#section-{{ section.id }}` scoping with `{% style %}` CSS variables block
- [ ] Full 4-part padding, desktop AND mobile separately (8 settings total)
- [ ] `section_max_width` range setting
- [ ] Background type selector (solid/gradient/image) with all sub-options including overlay
- [ ] Section borders — top AND bottom independently (enable, width, style, color)

**Typography:**
- [ ] Every text element has BOTH desktop AND mobile size settings
- [ ] Heading has: color, desktop size, mobile size, weight, line-height, transform, letter-spacing, margin-bottom
- [ ] Subtitle has: color, desktop size, mobile size, margin-bottom
- [ ] Accent text has: color, gradient option, font family override, font style, font weight

**Blocks & content:**
- [ ] All repeated content uses blocks (not hardcoded items)
- [ ] Section renders gracefully with zero blocks (empty state)
- [ ] CTA button (if present) has full style control (bg, text, border, radius, size, weight, padding, transform)

**Cards (if section has cards):**
- [ ] border-radius, border (width/style/color), shadow toggle
- [ ] 4-side margin control AND gap between cards
- [ ] Card padding (vertical + horizontal)

**Atlas/Elixir extras:**
- [ ] Animation setting (none/fade/slide-up/slide-left/zoom) with duration and delay
- [ ] Google Font override (heading font + body font selectors)
- [ ] `layout_gap` for column spacing
- [ ] `header_bottom_margin` for space between heading and content

**Mobile:**
- [ ] CSS is written mobile-first (base = mobile, `@media (min-width: 750px)` = desktop)
- [ ] No horizontal scroll on 390px viewport
- [ ] Touch targets ≥ 44px on mobile
- [ ] Font sizes ≥ 14px body, ≥ 22px headings on mobile
- [ ] Full-width CTAs on mobile

**Code quality:**
- [ ] No placeholder text, no unfilled variables, no TODO comments
- [ ] All colors come from CSS variables (set in `{% style %}` block), not hardcoded in HTML
