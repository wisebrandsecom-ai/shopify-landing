# Elixir Design Language — Reference

Loaded during Step 6 of the shopify-landing skill.
Defines the visual and technical standards Claude must apply when building custom sections.
Based on full analysis of the Elixir theme (88 sections, 120 CSS files).

**Priority rule:** Competitor replication always wins. Elixir patterns are the quality floor, not the ceiling. When competitor and Elixir conflict, follow the competitor. When competitor doesn't specify, apply Elixir's approach.

---

## 1. Heading system — two-part accent headings

Elixir's signature visual: every major heading has a **regular part** and an **accent part** in a different color, font, or style. This is the single most recognizable Elixir pattern.

### Pattern
```liquid
<h2 class="section-heading">
  <span class="heading-regular" style="color: {{ section.settings.heading_color }};">{{ section.settings.heading_regular }}</span>
  <span class="heading-accent" style="color: {{ section.settings.heading_accent_color }}; font-style: {{ section.settings.accent_italic | ternary: 'italic', 'normal' }};">{{ section.settings.heading_accent }}</span>
</h2>
```

### Schema for any section heading
```json
{ "type": "header", "content": "Heading" },
{ "type": "text",   "id": "heading_regular",      "label": "Heading — regular part",  "default": "The Simple Way To" },
{ "type": "text",   "id": "heading_accent",       "label": "Heading — accent part",   "default": "Breathe Again" },
{ "type": "color",  "id": "heading_color",        "label": "Regular text color",      "default": "#111111" },
{ "type": "color",  "id": "heading_accent_color", "label": "Accent text color",       "default": "#1a4731" },
{ "type": "checkbox","id": "accent_italic",        "label": "Italic accent",           "default": false },
{ "type": "range",  "id": "heading_size_desktop", "label": "Heading size desktop",    "min": 24, "max": 72, "step": 2, "unit": "px", "default": 42 },
{ "type": "range",  "id": "heading_size_mobile",  "label": "Heading size mobile",     "min": 20, "max": 48, "step": 2, "unit": "px", "default": 28 },
{ "type": "select", "id": "heading_weight",       "label": "Heading weight",
  "options": [{"value":"500","label":"Medium"},{"value":"600","label":"Semibold"},{"value":"700","label":"Bold"},{"value":"800","label":"Extrabold"}],
  "default": "700" },
{ "type": "select", "id": "heading_align",        "label": "Alignment",
  "options": [{"value":"left","label":"Left"},{"value":"center","label":"Center"}],
  "default": "center" }
```

### CSS
```css
.section-heading { line-height: 1.1; margin-bottom: 16px; }
.heading-accent { display: inline; }
@media (min-width: 750px) {
  .section-heading { margin-bottom: 24px; }
}
```

---

## 2. CSS variable system — inline style block per section

Elixir never uses hardcoded colors in CSS files. Every section uses a `{% style %}` block that sets CSS custom properties, then CSS uses those variables. This keeps sections isolated and theme-customizer-reactive.

### Pattern
```liquid
{%- style -%}
  #section-{{ section.id }} {
    --section-bg:        {{ section.settings.bg_color }};
    --section-text:      {{ section.settings.text_color }};
    --section-accent:    {{ section.settings.accent_color }};
    --heading-size:      {{ section.settings.heading_size_mobile }}px;
    --padding-top:       {{ section.settings.padding_top_mobile }}px;
    --padding-bottom:    {{ section.settings.padding_bottom_mobile }}px;
    --padding-h:         {{ section.settings.padding_h_mobile }}px;
  }
  @media (min-width: 750px) {
    #section-{{ section.id }} {
      --heading-size:    {{ section.settings.heading_size_desktop }}px;
      --padding-top:     {{ section.settings.padding_top }}px;
      --padding-bottom:  {{ section.settings.padding_bottom }}px;
      --padding-h:       {{ section.settings.padding_h }}px;
    }
  }
{%- endstyle -%}

<section id="section-{{ section.id }}" style="background: var(--section-bg); padding: var(--padding-top) var(--padding-h) var(--padding-bottom);">
  ...
</section>
```

**Apply this pattern to every section Claude creates.** Use `#section-{{ section.id }}` as the scoping selector to avoid class collisions between sections.

---

## 3. Card system — elevated cards with full control

Elixir's cards are always configurable. Every card-based section exposes: border-radius, border, shadow, background, padding.

### Card CSS pattern
```css
/* Mobile base */
.el-card {
  background: var(--card-bg);
  border-radius: var(--card-radius);
  border: var(--card-border-width) solid var(--card-border-color);
  box-shadow: var(--card-shadow);
  padding: var(--card-pad-v) var(--card-pad-h);
  overflow: hidden;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.el-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0,0,0,0.12);
}
```

### Card schema settings
```json
{ "type": "header", "content": "Card Style" },
{ "type": "color",  "id": "card_bg",           "label": "Card background",   "default": "#ffffff" },
{ "type": "range",  "id": "card_radius",        "label": "Border radius",     "min": 0, "max": 30, "step": 2, "unit": "px", "default": 12 },
{ "type": "checkbox","id": "card_border",       "label": "Show border",       "default": true },
{ "type": "range",  "id": "card_border_width",  "label": "Border width",      "min": 1, "max": 4,  "step": 1, "unit": "px", "default": 1 },
{ "type": "color",  "id": "card_border_color",  "label": "Border color",      "default": "#e5e7eb" },
{ "type": "checkbox","id": "card_shadow",       "label": "Show shadow",       "default": false },
{ "type": "range",  "id": "card_pad_v",         "label": "Card padding v",    "min": 12, "max": 40, "step": 4, "unit": "px", "default": 20 },
{ "type": "range",  "id": "card_pad_h",         "label": "Card padding h",    "min": 12, "max": 40, "step": 4, "unit": "px", "default": 20 },
{ "type": "range",  "id": "card_gap",           "label": "Gap between cards", "min": 8,  "max": 40, "step": 4, "unit": "px", "default": 16 }
```

---

## 4. Background system — solid / gradient / image

Every Elixir section with a significant background uses this three-way selector:

```json
{ "type": "select", "id": "bg_type", "label": "Background type",
  "options": [{"value":"solid","label":"Solid Color"},{"value":"gradient","label":"Gradient"},{"value":"image","label":"Image"}],
  "default": "solid" },
{ "type": "color",            "id": "bg_color",    "label": "Background color",   "default": "#ffffff" },
{ "type": "color_background", "id": "bg_gradient", "label": "Background gradient","default": "linear-gradient(135deg, #1a4731, #2d6b50)" },
{ "type": "image_picker",     "id": "bg_image",    "label": "Background image" },
{ "type": "select", "id": "bg_image_size",
  "options": [{"value":"cover","label":"Cover"},{"value":"contain","label":"Contain"}], "default": "cover" },
{ "type": "checkbox", "id": "bg_overlay",        "label": "Enable overlay",     "default": false },
{ "type": "color",    "id": "bg_overlay_color",  "label": "Overlay color",      "default": "#000000" },
{ "type": "range",    "id": "bg_overlay_opacity","label": "Overlay opacity",    "min": 0, "max": 0.8, "step": 0.05, "default": 0.3 }
```

Liquid implementation:
```liquid
{%- liquid
  case section.settings.bg_type
    when 'solid'
      assign bg_style = 'background-color: ' | append: section.settings.bg_color
    when 'gradient'
      assign bg_style = 'background: ' | append: section.settings.bg_gradient
    when 'image'
      if section.settings.bg_image != blank
        assign bg_style = 'background-image: url(' | append: section.settings.bg_image | image_url: width: 1600 | append: '); background-size: ' | append: section.settings.bg_image_size | append: '; background-position: center;'
      else
        assign bg_style = 'background-color: ' | append: section.settings.bg_color
      endif
  endcase
-%}
<section style="{{ bg_style }}; position: relative;">
  {%- if section.settings.bg_overlay and section.settings.bg_type == 'image' -%}
  <div style="position: absolute; inset: 0; background: {{ section.settings.bg_overlay_color }}; opacity: {{ section.settings.bg_overlay_opacity }};"></div>
  {%- endif -%}
  <div style="position: relative; z-index: 1;">
    ... content ...
  </div>
</section>
```

---

## 5. Image handling — square with padding-bottom trick

Elixir uses the `padding-bottom: 100%` trick for perfectly square images at any container width:

```css
.el-image-wrap {
  width: 100%;
  height: 0;
  padding-bottom: 100%; /* or 75% for 4:3, 56.25% for 16:9, 125% for 4:5 */
  position: relative;
  border-radius: var(--card-radius);
  overflow: hidden;
  background: #f5f5f5; /* placeholder color */
}
.el-image-wrap img {
  position: absolute;
  top: 0; left: 0;
  width: 100%; height: 100%;
  object-fit: cover;
  display: block;
}
```

Schema for aspect ratio:
```json
{ "type": "select", "id": "image_ratio", "label": "Image aspect ratio",
  "options": [
    {"value":"100%",    "label": "Square (1:1)"},
    {"value":"125%",    "label": "Portrait (4:5)"},
    {"value":"133.33%", "label": "Portrait (3:4)"},
    {"value":"75%",     "label": "Landscape (4:3)"},
    {"value":"56.25%",  "label": "Widescreen (16:9)"}
  ], "default": "100%" }
```

---

## 6. Navigation buttons (carousel) — circular arrow buttons

Used in reviews carousel, steps carousel, ingredients slider:

```css
.el-nav-btn {
  width: 40px; height: 40px;
  border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  cursor: pointer;
  transition: all 0.3s ease;
  border: 1px solid var(--btn-border);
  background: var(--btn-bg);
  color: var(--btn-color);
}
.el-nav-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 2px 8px rgba(0,0,0,0.12);
}
.el-nav-btn:disabled { opacity: 0.4; cursor: default; }
@media (max-width: 749px) {
  .el-nav-btn { width: 34px; height: 34px; }
}
```

---

## 7. Scrollable card rows — horizontal scroll on mobile

Reviews, ingredients, and video testimonials use horizontal scroll on mobile:

```css
.el-scroll-row {
  display: flex;
  gap: 16px;
  overflow-x: auto;
  scroll-behavior: smooth;
  -webkit-overflow-scrolling: touch;
  scrollbar-width: none;
  padding-bottom: 8px;
  align-items: flex-start;
}
.el-scroll-row::-webkit-scrollbar { display: none; }
.el-scroll-card {
  flex: 0 0 300px;
  min-width: 0;
}
@media (max-width: 480px) {
  .el-scroll-card { flex: 0 0 calc(100vw - 60px); max-width: 320px; min-width: 280px; }
}
@media (min-width: 750px) {
  .el-scroll-row { overflow-x: visible; flex-wrap: wrap; }
  .el-scroll-card { flex: 0 0 calc(33.333% - 12px); }
}
```

---

## 8. Section catalog — what each Elixir section does and how to replicate it

When building a section to match a competitor element, use the matching Elixir pattern as the quality reference:

| Competitor element | Elixir equivalent | Key visual pattern |
|---|---|---|
| Video testimonials with reviews | `video-testimonials` | Horizontal scroll cards, video thumbnail + play button overlay, star rating, reviewer name+age badge, verified checkmark |
| Benefits list with image | `product-benefits` | Two-column (image left OR right), square image with padding-bottom trick, benefit items with SVG icon + title + description |
| Timeline / results by week | `steps` | Numbered steps with accent color, progress cards, optional carousel, stat bars with percentage |
| Comparison table (vs competitors) | `new-comparison` | Two-column table, checkmark/X icons, brand column highlighted with accent color |
| Social proof statistics (93%, 90%) | `statistics-grid` | Large percentage numbers, circular progress indicators (optional), card grid |
| Money-back guarantee section | `satisfaction-guarantee` | Shield icon (configurable style), polaroid-style photos at angles, centered content |
| As seen in / logos bar | `as-seen-in-logos` | Grayscale logos, hover to color, configurable logo height/width, mobile responsive grid |
| Scrolling ticker bar | `scrolling-ticker` | CSS animation, configurable speed, divider type (text/image), block-based items |
| Countdown sale banner | `sale-countdown-banner` | CSS variables for all colors, JS countdown to fixed date or midnight reset, sticky option |
| Doctor/expert validation | `customer-reviews-carousel` | Expert photo + name + title + quote, card style, carousel on mobile |
| Ingredients carousel | `benefits-carousel` | Image + ingredient name + accent label + description, horizontal scroll |
| Before/after comparison | `before-after-comparison` | Slider handle, two images side by side or split panel |
| Alternating text+image | `alternating-features` | Alternates image left/right on desktop, stacks on mobile |

---

## 9. Verified buyer badge pattern

Used in review cards throughout Elixir:

```html
<span class="el-verified" style="
  display: inline-flex; align-items: center; gap: 3px;
  font-size: 11px; font-weight: 600;
  color: VAR_VERIFIED_COLOR;
">
  <svg viewBox="0 0 24 24" width="12" height="12" fill="VAR_VERIFIED_COLOR">
    <path d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"/>
  </svg>
  Comprador Verificado
</span>
```

---

## 10. Review card — full Elixir pattern

```html
<div class="el-review-card" style="
  background: var(--card-bg);
  border-radius: var(--card-radius);
  border: 1px solid var(--card-border);
  overflow: hidden;
  display: flex; flex-direction: column;
">
  <!-- Optional video/image thumbnail -->
  <div class="el-review-image" style="width:100%; aspect-ratio:4/3; background:#1a4731; position:relative;">
    <!-- play button overlay for video -->
    <div style="position:absolute;inset:0;display:flex;align-items:center;justify-content:center;">
      <div style="width:48px;height:48px;border-radius:50%;background:rgba(255,255,255,0.9);display:flex;align-items:center;justify-content:center;">
        <svg viewBox="0 0 24 24" width="20" height="20" fill="#111"><polygon points="5 3 19 12 5 21 5 3"/></svg>
      </div>
    </div>
  </div>
  <!-- Review content -->
  <div style="padding:16px;flex:1;display:flex;flex-direction:column;gap:8px;">
    <div style="color:STAR_COLOR;font-size:14px;letter-spacing:1px;">★★★★★</div>
    <p style="font-size:14px;line-height:1.6;color:TEXT_COLOR;font-style:italic;margin:0;">"REVIEW_QUOTE"</p>
    <div style="display:flex;align-items:center;gap:8px;flex-wrap:wrap;">
      <span style="font-weight:600;font-size:13px;color:TEXT_COLOR;">REVIEWER_NAME</span>
      <span style="color:MUTED_COLOR;font-size:12px;">· AGE</span>
      [verified badge]
    </div>
    <!-- Optional tags -->
    <div style="display:flex;gap:6px;flex-wrap:wrap;">
      <span style="background:TAG_BG;color:TAG_COLOR;border-radius:20px;padding:2px 10px;font-size:11px;">TAG_1</span>
    </div>
  </div>
</div>
```

---

## 11. Section spacing standards (Elixir defaults)

| Element | Mobile | Desktop |
|---|---|---|
| Section padding top/bottom | 32px | 60px |
| Section padding horizontal | 20px | 40px (or 5rem) |
| Heading margin bottom | 12px | 24px |
| Subheading margin bottom | 20px | 32px |
| Card gap | 12px | 20px |
| Between section elements | 16px | 24px |
| Max content width | 100% | 1200px |

---

## 12. The `use_theme_colors` toggle — always include

Every Elixir section has this toggle at the top. When ON, it pulls global theme colors instead of per-section colors:

```liquid
{%- if section.settings.use_theme_colors -%}
  {%- assign bg = settings.global_section_background_color -%}
  {%- assign text = settings.global_section_text_color -%}
  {%- assign accent = settings.global_section_accent_1_color -%}
{%- else -%}
  {%- assign bg = section.settings.bg_color -%}
  {%- assign text = section.settings.text_color -%}
  {%- assign accent = section.settings.accent_color -%}
{%- endif -%}
```

---

## 13. Step/timeline section — Elixir pattern

Used for "results by week" timelines common in DTC health products:

```html
<div class="el-step" style="
  background: var(--step-card-bg);
  border-radius: var(--card-radius);
  border: 1px solid var(--step-border-color);
  padding: 16px 20px;
  position: relative;
">
  <!-- Step number badge -->
  <div style="
    display:inline-flex; align-items:center; gap:6px;
    font-size:12px; font-weight:600;
    color: var(--step-number-color);
    margin-bottom: 8px;
  ">
    <svg viewBox="0 0 24 24" width="14" height="14" fill="none" stroke="currentColor" stroke-width="2">
      <circle cx="12" cy="12" r="10"/>
      <polyline points="12 6 12 12 16 14"/>
    </svg>
    STEP_LABEL
  </div>
  <!-- Step title -->
  <div style="font-size:15px; font-weight:700; color:var(--step-title-color); margin-bottom:6px;">STEP_TITLE</div>
  <!-- Checkmarks -->
  <ul style="list-style:none;padding:0;margin:0;display:flex;flex-direction:column;gap:4px;">
    <li style="display:flex;align-items:flex-start;gap:6px;font-size:13px;color:var(--step-text-color);">
      <svg viewBox="0 0 24 24" width="14" height="14" fill="none" stroke="var(--accent)" stroke-width="2.5" style="flex-shrink:0;margin-top:2px;"><polyline points="20 6 9 17 4 12"/></svg>
      CHECKMARK_TEXT
    </li>
  </ul>
  <!-- Stat bar -->
  <div style="
    background: var(--stat-bar-bg);
    border-radius:4px; padding: 6px 10px;
    font-size:12px; font-weight:600;
    color: var(--stat-bar-text);
    margin-top:10px;
  ">STAT_TEXT</div>
</div>
```

---

## 14. Application rules for Claude

When building any custom section:

1. **Always use** `#section-{{ section.id }}` scoping with CSS custom properties (`{% style %}` block)
2. **Always use** two-part headings (regular + accent color) for section titles
3. **Always use** the background type selector (solid/gradient/image)
4. **Always include** `use_theme_colors` toggle
5. **Always write** CSS mobile-first (`base = mobile`, `@media (min-width: 750px) = desktop`)
6. **Cards** always get: radius, border toggle, shadow toggle, padding controls
7. **Image containers** always use the `padding-bottom` aspect ratio trick
8. **Mobile scrollable rows** always use the `overflow-x: auto; scrollbar-width: none` pattern
9. **Navigation buttons** (if carousel) always get the circular button pattern with hover lift
10. **Review cards** always include: stars, quote, reviewer name+age, verified badge, optional tags


---

## 15. Full micro-customization schema — extracted from Elixir sections

Every section Claude generates must expose these setting groups, modelled on Elixir's actual schemas. This is what separates a professional section from a basic one.

### 15.1 Section padding — desktop + mobile separate

```json
{ "type": "header", "content": "Section Padding — Desktop" },
{ "type": "range", "id": "padding_top",    "label": "Top Padding",    "min": 0, "max": 120, "step": 4, "unit": "px", "default": 60 },
{ "type": "range", "id": "padding_bottom", "label": "Bottom Padding", "min": 0, "max": 120, "step": 4, "unit": "px", "default": 60 },
{ "type": "range", "id": "padding_left",   "label": "Left Padding",   "min": 0, "max": 80,  "step": 4, "unit": "px", "default": 20 },
{ "type": "range", "id": "padding_right",  "label": "Right Padding",  "min": 0, "max": 80,  "step": 4, "unit": "px", "default": 20 },
{ "type": "header", "content": "Section Padding — Mobile" },
{ "type": "range", "id": "padding_top_mobile",    "label": "Top Padding (Mobile)",    "min": 0, "max": 80, "step": 4, "unit": "px", "default": 32 },
{ "type": "range", "id": "padding_bottom_mobile", "label": "Bottom Padding (Mobile)", "min": 0, "max": 80, "step": 4, "unit": "px", "default": 32 },
{ "type": "range", "id": "padding_horizontal_mobile", "label": "Horizontal Padding (Mobile)", "min": 0, "max": 40, "step": 4, "unit": "px", "default": 20 }
```

### 15.2 Section borders — top and bottom independently

```json
{ "type": "header", "content": "Section Borders" },
{ "type": "checkbox", "id": "enable_top_border",    "label": "Enable Top Border",    "default": false },
{ "type": "range",   "id": "top_border_width",      "label": "Top Border Width",      "min": 1, "max": 8, "step": 1, "unit": "px", "default": 1 },
{ "type": "select",  "id": "top_border_style",      "label": "Top Border Style",
  "options": [{"value":"solid","label":"Solid"},{"value":"dashed","label":"Dashed"},{"value":"dotted","label":"Dotted"},{"value":"double","label":"Double"}],
  "default": "solid" },
{ "type": "color",   "id": "top_border_color",      "label": "Top Border Color",      "default": "#e5e7eb" },
{ "type": "checkbox", "id": "enable_bottom_border", "label": "Enable Bottom Border",  "default": false },
{ "type": "range",   "id": "bottom_border_width",   "label": "Bottom Border Width",   "min": 1, "max": 8, "step": 1, "unit": "px", "default": 1 },
{ "type": "select",  "id": "bottom_border_style",   "label": "Bottom Border Style",
  "options": [{"value":"solid","label":"Solid"},{"value":"dashed","label":"Dashed"},{"value":"dotted","label":"Dotted"},{"value":"double","label":"Double"}],
  "default": "solid" },
{ "type": "color",   "id": "bottom_border_color",   "label": "Bottom Border Color",   "default": "#e5e7eb" }
```

### 15.3 Container border + shadow (for contained sections)

```json
{ "type": "header",   "content": "Container Border & Effects" },
{ "type": "checkbox", "id": "enable_border",    "label": "Enable Border",    "default": false },
{ "type": "range",    "id": "border_width",     "label": "Border Width",     "min": 1, "max": 8, "step": 1, "unit": "px", "default": 1 },
{ "type": "select",   "id": "border_style",     "label": "Border Style",
  "options": [{"value":"solid","label":"Solid"},{"value":"dashed","label":"Dashed"},{"value":"dotted","label":"Dotted"},{"value":"double","label":"Double"}],
  "default": "solid" },
{ "type": "color",    "id": "border_color",     "label": "Border Color",     "default": "#e5e7eb" },
{ "type": "range",    "id": "border_radius",    "label": "Border Radius",    "min": 0, "max": 40, "step": 2, "unit": "px", "default": 0 },
{ "type": "checkbox", "id": "enable_shadow",    "label": "Enable Shadow",    "default": false },
{ "type": "range",    "id": "shadow_opacity",   "label": "Shadow Opacity",   "min": 0, "max": 0.5, "step": 0.05, "default": 0.1 }
```

### 15.4 Typography — full control per text element (Elixir pattern)

Every distinct text element gets: color, desktop size, mobile size, weight. Headings also get: line height, letter spacing, transform.

```json
{ "type": "header", "content": "Heading Typography" },
{ "type": "color",  "id": "heading_color",        "label": "Heading Color",         "default": "#111111" },
{ "type": "range",  "id": "heading_size_desktop",  "label": "Heading Size (Desktop)","min": 20, "max": 72, "step": 2, "unit": "px", "default": 40 },
{ "type": "range",  "id": "heading_size_mobile",   "label": "Heading Size (Mobile)", "min": 18, "max": 48, "step": 2, "unit": "px", "default": 26 },
{ "type": "range",  "id": "heading_line_height",   "label": "Heading Line Height",   "min": 1,  "max": 2,  "step": 0.05, "default": 1.15 },
{ "type": "select", "id": "heading_weight",        "label": "Heading Font Weight",
  "options": [{"value":"400","label":"Regular"},{"value":"500","label":"Medium"},{"value":"600","label":"Semibold"},{"value":"700","label":"Bold"},{"value":"800","label":"Extrabold"}],
  "default": "700" },
{ "type": "select", "id": "heading_transform",     "label": "Text Transform",
  "options": [{"value":"none","label":"None"},{"value":"uppercase","label":"Uppercase"},{"value":"capitalize","label":"Capitalize"}],
  "default": "none" },
{ "type": "range",  "id": "heading_letter_spacing","label": "Letter Spacing",        "min": -2, "max": 8, "step": 0.5, "unit": "px", "default": 0 },
{ "type": "range",  "id": "heading_margin_bottom", "label": "Heading Margin Bottom", "min": 0,  "max": 60, "step": 2, "unit": "px", "default": 16 },

{ "type": "header", "content": "Subtitle Typography" },
{ "type": "color",  "id": "subtitle_color",        "label": "Subtitle Color",        "default": "#555555" },
{ "type": "range",  "id": "subtitle_size_desktop", "label": "Subtitle Size (Desktop)","min": 12, "max": 28, "step": 1, "unit": "px", "default": 16 },
{ "type": "range",  "id": "subtitle_size_mobile",  "label": "Subtitle Size (Mobile)", "min": 12, "max": 22, "step": 1, "unit": "px", "default": 14 },
{ "type": "range",  "id": "subtitle_margin_bottom","label": "Subtitle Margin Bottom", "min": 0,  "max": 60, "step": 2, "unit": "px", "default": 24 },

{ "type": "header", "content": "Body Text Typography" },
{ "type": "color",  "id": "body_color",            "label": "Body Text Color",       "default": "#374151" },
{ "type": "range",  "id": "body_size_desktop",     "label": "Body Size (Desktop)",   "min": 12, "max": 22, "step": 1, "unit": "px", "default": 16 },
{ "type": "range",  "id": "body_size_mobile",      "label": "Body Size (Mobile)",    "min": 12, "max": 18, "step": 1, "unit": "px", "default": 14 },
{ "type": "range",  "id": "body_line_height",      "label": "Line Height",           "min": 1,  "max": 2.5, "step": 0.1, "default": 1.6 }
```

### 15.5 Accent text system (Elixir's signature)

```json
{ "type": "header",   "content": "Accent Text" },
{ "type": "color",    "id": "accent_color",           "label": "Accent Color",          "default": "#1a4731" },
{ "type": "checkbox", "id": "use_accent_gradient",    "label": "Use Accent Gradient",   "default": false },
{ "type": "color_background", "id": "accent_gradient","label": "Accent Gradient",       "default": "linear-gradient(135deg, #1a4731, #2d9b6f)" },
{ "type": "range",    "id": "accent_text_margin_left","label": "Accent Text Left Margin","min": 0, "max": 20, "step": 1, "unit": "px", "default": 0, "info": "Space between regular and accent part of heading" },
{ "type": "header",   "content": "Accent Font (override global)" },
{ "type": "checkbox", "id": "override_global_accent", "label": "Override Global Accent Font", "default": false },
{ "type": "select",   "id": "accent_font_family",     "label": "Accent Font Family",
  "options": [
    {"value":"inherit",         "label":"Same as body"},
    {"value":"serif",           "label":"System Serif"},
    {"value":"Bodoni Moda",     "label":"Bodoni Moda (elegant)"},
    {"value":"Playfair Display","label":"Playfair Display (editorial)"},
    {"value":"Cormorant Garamond","label":"Cormorant Garamond (luxury)"},
    {"value":"DM Serif Text",   "label":"DM Serif Text (modern serif)"},
    {"value":"Instrument Serif","label":"Instrument Serif (clean serif)"},
    {"value":"Nothing You Could Do","label":"Nothing You Could Do (handwritten)"}
  ], "default": "inherit" },
{ "type": "select",   "id": "accent_font_style",      "label": "Accent Font Style",
  "options": [{"value":"normal","label":"Normal"},{"value":"italic","label":"Italic"}],
  "default": "normal" },
{ "type": "select",   "id": "accent_font_weight",     "label": "Accent Font Weight",
  "options": [{"value":"300","label":"Light"},{"value":"400","label":"Regular"},{"value":"500","label":"Medium"},{"value":"600","label":"Semibold"},{"value":"700","label":"Bold"}],
  "default": "400" }
```

Liquid implementation for gradient accent:
```liquid
{%- if section.settings.use_accent_gradient -%}
  {%- assign accent_style = 'background: ' | append: section.settings.accent_gradient | append: '; -webkit-background-clip: text; -webkit-text-fill-color: transparent; background-clip: text;' -%}
{%- else -%}
  {%- assign accent_style = 'color: ' | append: section.settings.accent_color -%}
{%- endif -%}
<span class="heading-accent" style="{{ accent_style }}; margin-left: {{ section.settings.accent_text_margin_left }}px;">
  {{ section.settings.heading_accent }}
</span>
```

### 15.6 Card margin control (4 sides individually)

```json
{ "type": "header", "content": "Card Spacing" },
{ "type": "range",  "id": "card_margin_top",    "label": "Card Top Margin",    "min": 0, "max": 40, "step": 2, "unit": "px", "default": 0 },
{ "type": "range",  "id": "card_margin_right",  "label": "Card Right Margin",  "min": 0, "max": 40, "step": 2, "unit": "px", "default": 0 },
{ "type": "range",  "id": "card_margin_bottom", "label": "Card Bottom Margin", "min": 0, "max": 40, "step": 2, "unit": "px", "default": 0 },
{ "type": "range",  "id": "card_margin_left",   "label": "Card Left Margin",   "min": 0, "max": 40, "step": 2, "unit": "px", "default": 0 },
{ "type": "range",  "id": "card_gap",           "label": "Gap Between Cards",  "min": 8, "max": 48, "step": 4, "unit": "px", "default": 16 }
```

### 15.7 Section content width + layout gap

```json
{ "type": "header", "content": "Layout" },
{ "type": "range",  "id": "section_max_width", "label": "Content Max Width",       "min": 600, "max": 1600, "step": 20, "unit": "px", "default": 1200 },
{ "type": "range",  "id": "layout_gap",        "label": "Gap Between Columns",     "min": 0,   "max": 80,   "step": 4,  "unit": "px", "default": 40 },
{ "type": "range",  "id": "header_bottom_margin","label": "Header Bottom Margin",  "min": 0,   "max": 80,   "step": 4,  "unit": "px", "default": 32 }
```

---

## 16. Google Fonts system (Atlas-inspired)

Every section with heading text should support Google Font overrides. This is what prevents the "React/generic" look.

### Schema
```json
{ "type": "header",  "content": "Fonts" },
{ "type": "select",  "id": "heading_font",  "label": "Heading Font",
  "options": [
    {"value":"inherit",           "label":"Theme default"},
    {"value":"Playfair Display",  "label":"Playfair Display (editorial)"},
    {"value":"Cormorant Garamond","label":"Cormorant Garamond (luxury)"},
    {"value":"Bodoni Moda",       "label":"Bodoni Moda (fashion)"},
    {"value":"DM Serif Display",  "label":"DM Serif Display (modern)"},
    {"value":"Sora",              "label":"Sora (clean tech)"},
    {"value":"Outfit",            "label":"Outfit (modern sans)"},
    {"value":"Plus Jakarta Sans", "label":"Plus Jakarta Sans (professional)"},
    {"value":"Space Grotesk",     "label":"Space Grotesk (geometric)"},
    {"value":"Manrope",           "label":"Manrope (friendly sans)"}
  ], "default": "inherit" },
{ "type": "select",  "id": "body_font",    "label": "Body Font",
  "options": [
    {"value":"inherit",          "label":"Theme default"},
    {"value":"Inter",            "label":"Inter (clean)"},
    {"value":"Outfit",           "label":"Outfit (modern)"},
    {"value":"Plus Jakarta Sans","label":"Plus Jakarta Sans"},
    {"value":"DM Sans",          "label":"DM Sans (friendly)"},
    {"value":"Nunito",           "label":"Nunito (rounded)"},
    {"value":"Lato",             "label":"Lato (classic)"}
  ], "default": "inherit" }
```

### Liquid implementation
```liquid
{%- if section.settings.heading_font != 'inherit' or section.settings.body_font != 'inherit' -%}
  {%- assign fonts_to_load = '' -%}
  {%- if section.settings.heading_font != 'inherit' -%}
    {%- assign heading_font_encoded = section.settings.heading_font | replace: ' ', '+' -%}
    {%- assign fonts_to_load = fonts_to_load | append: heading_font_encoded | append: ':wght@400;600;700;800&' -%}
  {%- endif -%}
  {%- if section.settings.body_font != 'inherit' and section.settings.body_font != section.settings.heading_font -%}
    {%- assign body_font_encoded = section.settings.body_font | replace: ' ', '+' -%}
    {%- assign fonts_to_load = fonts_to_load | append: body_font_encoded | append: ':wght@300;400;500;600&' -%}
  {%- endif -%}
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family={{ fonts_to_load }}display=swap" rel="stylesheet">
{%- endif -%}

{%- style -%}
  #section-{{ section.id }} {
    {%- if section.settings.heading_font != 'inherit' -%}
    --section-heading-font: '{{ section.settings.heading_font }}', var(--font-heading-family);
    {%- endif -%}
    {%- if section.settings.body_font != 'inherit' -%}
    --section-body-font: '{{ section.settings.body_font }}', var(--font-body-family);
    {%- endif -%}
  }
  #section-{{ section.id }} h1,
  #section-{{ section.id }} h2,
  #section-{{ section.id }} h3 {
    font-family: var(--section-heading-font, var(--font-heading-family));
  }
  #section-{{ section.id }} p,
  #section-{{ section.id }} li,
  #section-{{ section.id }} span {
    font-family: var(--section-body-font, var(--font-body-family));
  }
{%- endstyle -%}
```

---

## 17. CSS entrance animations (Atlas-inspired)

Sections should have optional fade-in/slide-up animations as elements enter the viewport. Uses Intersection Observer — no external libraries.

### Schema setting
```json
{ "type": "header",  "content": "Animation" },
{ "type": "select",  "id": "animation_type", "label": "Entrance Animation",
  "options": [
    {"value":"none",     "label":"None"},
    {"value":"fade",     "label":"Fade In"},
    {"value":"slide-up", "label":"Slide Up"},
    {"value":"slide-left","label":"Slide In Left"},
    {"value":"zoom",     "label":"Zoom In"}
  ], "default": "none" },
{ "type": "range",   "id": "animation_duration","label": "Animation Duration (ms)","min": 200, "max": 1200, "step": 100, "default": 600 },
{ "type": "range",   "id": "animation_delay",   "label": "Animation Delay (ms)",  "min": 0,   "max": 800,  "step": 50,  "default": 0 }
```

### Liquid + CSS + JS implementation
```liquid
{%- assign anim = section.settings.animation_type -%}
{%- if anim != 'none' -%}
<style>
  #section-{{ section.id }} [data-animate] {
    opacity: 0;
    transition: opacity {{ section.settings.animation_duration }}ms ease {{ section.settings.animation_delay }}ms,
                transform {{ section.settings.animation_duration }}ms ease {{ section.settings.animation_delay }}ms;
    {%- case anim -%}
    {%- when 'slide-up' -%}   transform: translateY(32px);
    {%- when 'slide-left' -%} transform: translateX(-32px);
    {%- when 'zoom' -%}        transform: scale(0.92);
    {%- endcase -%}
  }
  #section-{{ section.id }} [data-animate].is-visible {
    opacity: 1;
    transform: none;
  }
  @media (prefers-reduced-motion: reduce) {
    #section-{{ section.id }} [data-animate] { opacity: 1; transform: none; transition: none; }
  }
</style>
<script>
(function() {
  var observer = new IntersectionObserver(function(entries) {
    entries.forEach(function(entry) {
      if (entry.isIntersecting) {
        entry.target.classList.add('is-visible');
        observer.unobserve(entry.target);
      }
    });
  }, { threshold: 0.15 });
  document.querySelectorAll('#section-{{ section.id }} [data-animate]').forEach(function(el) {
    observer.observe(el);
  });
})();
</script>
{%- endif -%}
```

Add `data-animate` attribute to the elements that should animate:
```liquid
<div class="section-content" data-animate>
  ...heading, cards, etc...
</div>
```

For staggered card animations (each card delays slightly):
```liquid
{%- for block in section.blocks -%}
<div class="card" data-animate style="transition-delay: {{ forloop.index0 | times: 100 | plus: section.settings.animation_delay }}ms;">
  ...
</div>
{%- endfor -%}
```

---

## 18. Reviewer info border (Elixir micro-detail)

The reviewer badge in review cards can have its own border:

```json
{ "type": "header",   "content": "Reviewer Badge Border" },
{ "type": "checkbox", "id": "enable_reviewer_border",    "label": "Enable Border",   "default": false },
{ "type": "range",    "id": "reviewer_border_width",     "label": "Border Width",    "min": 1, "max": 4, "step": 1, "unit": "px", "default": 1 },
{ "type": "select",   "id": "reviewer_border_style",     "label": "Border Style",
  "options": [{"value":"solid","label":"Solid"},{"value":"dashed","label":"Dashed"},{"value":"dotted","label":"Dotted"}],
  "default": "solid" },
{ "type": "color",    "id": "reviewer_border_color",     "label": "Border Color",    "default": "#e5e7eb" },
{ "type": "range",    "id": "reviewer_border_radius",    "label": "Border Radius",   "min": 0, "max": 50, "step": 2, "unit": "px", "default": 20 }
```

---

## 19. Buy box drag & drop — how it works

The `custom-product.liquid` section already supports drag-and-drop block reordering **natively in the Shopify theme customizer**. No extra implementation needed.

**How to use:**
1. Go to Shopify Admin → Online Store → Themes → Customize
2. Click on the product page
3. In the left panel, click on the "Custom Product" section
4. All blocks (Rating, Title, Price, Pills, ATC, Badges, FAQ, etc.) appear as a list
5. Drag them to reorder — changes apply instantly

**For Katching Bundles:** Add the Katching Bundles app block directly in the customizer. It will appear as a block that can be dragged into the correct position within the custom-product section, between the benefit pills and the ATC button.

**Block order recommendation for DTC health products:**
```
1. [Rating]          ← stars + review count
2. [Social Proof]    ← "89% of customers:"
3. [Title]           ← product name
4. [Benefit Bullets] ← checkmark list
5. [Price]           ← with save badge
6. [Benefit Pills]   ← small tags
7. [Shipping Notice] ← "Ready to ship..."
8. [Katching Bundles]← app block (drag here)
9. [Above ATC]       ← guarantee strip
10. [Add to Cart]    ← ATC button
11. [Dynamic Checkout]← PayPal etc
12. [Payment Icons]
13. [Trust Badges]   ← 3-column
14. [Guarantee Badge]← shield + text
15. [Featured Review]← star quote
16. [FAQ]            ← accordion
```

