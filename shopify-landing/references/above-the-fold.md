# Above The Fold — Reference

Loaded during Step 6 of the shopify-landing skill.

---

## Core principle — READ THIS FIRST

**Build `sections/custom-product.liquid` from scratch. This applies to ALL themes.**

- Dawn → custom-product (remove main-product from template)
- Horizon → custom-product (remove product-information from template)
- Elixir → custom-product (remove shop-product-details from template)
- Any other theme → custom-product (remove whatever the native buy box is)

**The template JSON `order` array contains ONLY `custom-product`. Nothing else.**

**NEVER** keep the native buy box section in the template.
**NEVER** add buy box content (ATC, price, benefits, FAQ) as separate page sections below the fold.
**NEVER** say "I'll use the native section as the hero and add custom sections below."

The custom-product section contains its OWN gallery and its OWN buy box in one file.
The ATC button works because it uses `{% form 'product', product %}` natively.

This approach is inspired by how Elixir's `shop-product-details` section works: a fully block-based custom section that handles the entire right column of the product page, with granular schema settings for every element.

**Never add `variant_selector`, `quantity_break`, or `subscribe_save` blocks.** Katching Bundles handles this.

**Color contrast is mandatory.** Dark background → light text. Light background → dark text. Never same-hue. Min 4.5:1 contrast ratio.

**No placeholder comments in .liquid files. NEVER use `{# ... #}` syntax** — Shopify Liquid does not support it and renders it as visible text on the page. Use `{% comment %}...{% endcomment %}` or remove comments entirely. Never leave `{# ... #}`, `<!-- placeholder -->`, `[IMAGE HERE]`, or any unfilled placeholder in any file. Every section of the file must be complete working code before uploading.

**Remove main-product from template when using custom-product.** The template JSON `order` array must contain ONLY `custom-product`. If `main-product` is also present, both render simultaneously causing content overlap. Remove it.

**ABSOLUTE RULE — NO EMOJIS unless competitor uses them:**
- Default behavior: ALWAYS use inline SVG icons. Never use emojis as a substitute for icons.
- ONLY use emojis if the competitor explicitly uses emoji characters in that element.
- Check each element individually: trust badges, benefit pills, bullets, announcement bar, guarantee badge.
- When in doubt: use SVG.

SVG icon library for common DTC elements (use these, do not invent others):
```
Shield/guarantee:  <svg viewBox="0 0 24 24" width="SIZE" height="SIZE" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>
Truck/shipping:    <svg viewBox="0 0 24 24" width="SIZE" height="SIZE" fill="none" stroke="currentColor" stroke-width="2"><rect x="1" y="3" width="15" height="13"/><polygon points="16 8 20 8 23 11 23 16 16 16 16 8"/><circle cx="5.5" cy="18.5" r="2.5"/><circle cx="18.5" cy="18.5" r="2.5"/></svg>
Return/refund:     <svg viewBox="0 0 24 24" width="SIZE" height="SIZE" fill="none" stroke="currentColor" stroke-width="2"><polyline points="1 4 1 10 7 10"/><path d="M3.51 15a9 9 0 1 0 .49-3.5"/></svg>
Checkmark circle:  <svg viewBox="0 0 24 24" width="SIZE" height="SIZE" fill="none" stroke="currentColor" stroke-width="2"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><polyline points="22 4 12 14.01 9 11.01"/></svg>
Star/rating:       <svg viewBox="0 0 24 24" width="SIZE" height="SIZE" fill="currentColor"><polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"/></svg>
Leaf/natural:      <svg viewBox="0 0 24 24" width="SIZE" height="SIZE" fill="none" stroke="currentColor" stroke-width="2"><path d="M2 22c0-6.08 3.66-11.26 9-13.4A13 13 0 0 1 22 22H2z"/><path d="M12 8.5V22"/></svg>
Lock/secure:       <svg viewBox="0 0 24 24" width="SIZE" height="SIZE" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/></svg>
Clock/time:        <svg viewBox="0 0 24 24" width="SIZE" height="SIZE" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>
Flask/lab:         <svg viewBox="0 0 24 24" width="SIZE" height="SIZE" fill="none" stroke="currentColor" stroke-width="2"><path d="M9 3h6M9 3v8l-4 9h14L15 11V3M9 3h6"/></svg>
Heart/health:      <svg viewBox="0 0 24 24" width="SIZE" height="SIZE" fill="none" stroke="currentColor" stroke-width="2"><path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"/></svg>
```

Replace SIZE with the actual pixel value needed (e.g. 16, 20, 24).

---

## Step 1 — Read the theme to understand its structure

Before writing any code, pull these files:

```bash
shopify theme pull --only sections/main-product.liquid --theme WORKING_THEME_ID
shopify theme pull --only layout/theme.liquid --theme WORKING_THEME_ID
shopify theme pull --only templates/product.TEMPLATE_NAME.json --theme WORKING_THEME_ID
```

From `main-product.liquid`, identify:
- The CSS class of the product gallery wrapper (e.g. `.product__media-wrapper`, `.product-media`)
- Whether the theme has a media gallery snippet (e.g. `{% render 'product-media-gallery' %}`)
- The overall page layout (two-column vs stacked)

From `theme.liquid`, confirm how assets are loaded.

---

## Step 2 — Build `sections/custom-product.liquid`

This is the main deliverable. It is a complete two-column product section:
- **Left column:** product gallery (reuses theme's native gallery snippet if available, otherwise builds a simple image gallery)
- **Right column:** fully custom buy box with all persuasive elements, driven by schema blocks

### File structure

```
sections/
  custom-product.liquid    ← main section file
assets/
  custom-product.css       ← section styles
  custom-product.js        ← gallery JS if needed
```

### Full `sections/custom-product.liquid`

```liquid
{{ 'custom-product.css' | asset_url | stylesheet_tag }}

{%- assign cp = product -%}
{%- assign current_variant = cp.selected_or_first_available_variant -%}

<div class="cp-wrapper" style="background: {{ section.settings.page_bg }};">
  <div class="cp-container">

    {# ═══════════════════════════════════════ #}
    {# LEFT COLUMN — GALLERY                  #}
    {# ═══════════════════════════════════════ #}
    <div class="cp-gallery">

      {# Main image #}
      <div class="cp-main-image" id="cp-main-image">
        {%- if cp.featured_media -%}
          <img
            src="{{ cp.featured_media | image_url: width: 800 }}"
            alt="{{ cp.featured_media.alt | escape }}"
            width="800"
            height="{{ 800 | divided_by: cp.featured_media.aspect_ratio | round }}"
            loading="eager"
            id="cp-main-img"
            style="border-radius: {{ section.settings.image_border_radius }}px;"
          >
        {%- endif -%}
      </div>

      {# Thumbnails #}
      {%- if cp.media.size > 1 -%}
      <div class="cp-thumbnails">
        {%- for media in cp.media limit: 8 -%}
        <button
          class="cp-thumb{% if forloop.first %} cp-thumb--active{% endif %}"
          data-src="{{ media | image_url: width: 800 }}"
          data-index="{{ forloop.index0 }}"
          style="border-radius: {{ section.settings.image_border_radius }}px;"
        >
          <img
            src="{{ media | image_url: width: 120 }}"
            alt="{{ media.alt | escape }}"
            width="120"
            height="120"
            loading="lazy"
          >
        </button>
        {%- endfor -%}
      </div>
      {%- endif -%}

    </div>

    {# ═══════════════════════════════════════ #}
    {# RIGHT COLUMN — BUY BOX                 #}
    {# ═══════════════════════════════════════ #}
    <div class="cp-buybox">

      {%- for block in section.blocks -%}

        {# RATING #}
        {%- if block.type == 'rating' -%}
        <div class="cp-rating" style="margin-bottom: {{ block.settings.margin_bottom }}px;" {{ block.shopify_attributes }}>
          <span class="cp-stars" style="color: {{ block.settings.star_color }}; font-size: {{ block.settings.star_size }}px;">★★★★★</span>
          <span style="color: {{ block.settings.text_color }}; font-size: {{ block.settings.font_size }}px; font-weight: {{ block.settings.font_weight }};">
            {{ block.settings.text }}
          </span>
        </div>

        {# SOCIAL PROOF LINE #}
        {%- elsif block.type == 'social_proof' -%}
        <div class="cp-social-proof" style="color: {{ block.settings.color }}; font-size: {{ block.settings.font_size }}px; font-weight: {{ block.settings.font_weight }}; margin-bottom: {{ block.settings.margin_bottom }}px;" {{ block.shopify_attributes }}>
          {{ block.settings.text }}
        </div>

        {# TITLE #}
        {%- elsif block.type == 'title' -%}
        <h1 class="cp-title" style="color: {{ block.settings.color }}; font-size: {{ block.settings.font_size }}px; line-height: {{ block.settings.line_height }}; font-weight: {{ block.settings.font_weight }}; margin-bottom: {{ block.settings.margin_bottom }}px;" {{ block.shopify_attributes }}>
          {{ cp.title }}
        </h1>

        {# SUBTITLE #}
        {%- elsif block.type == 'subtitle' -%}
        {%- if block.settings.text != blank -%}
        <div class="cp-subtitle" style="color: {{ block.settings.color }}; font-size: {{ block.settings.font_size }}px; margin-bottom: {{ block.settings.margin_bottom }}px;" {{ block.shopify_attributes }}>
          {{ block.settings.text }}
        </div>
        {%- endif -%}

        {# BENEFIT BULLETS #}
        {%- elsif block.type == 'benefit_bullets' -%}
        <ul class="cp-bullets" style="margin-bottom: {{ block.settings.margin_bottom }}px; gap: {{ block.settings.gap }}px;" {{ block.shopify_attributes }}>
          {%- if block.settings.bullet_1 != blank -%}<li style="color: {{ block.settings.text_color }}; font-size: {{ block.settings.font_size }}px;"><span style="color: {{ block.settings.icon_color }};">{{ block.settings.icon }}</span> {{ block.settings.bullet_1 }}</li>{%- endif -%}
          {%- if block.settings.bullet_2 != blank -%}<li style="color: {{ block.settings.text_color }}; font-size: {{ block.settings.font_size }}px;"><span style="color: {{ block.settings.icon_color }};">{{ block.settings.icon }}</span> {{ block.settings.bullet_2 }}</li>{%- endif -%}
          {%- if block.settings.bullet_3 != blank -%}<li style="color: {{ block.settings.text_color }}; font-size: {{ block.settings.font_size }}px;"><span style="color: {{ block.settings.icon_color }};">{{ block.settings.icon }}</span> {{ block.settings.bullet_3 }}</li>{%- endif -%}
          {%- if block.settings.bullet_4 != blank -%}<li style="color: {{ block.settings.text_color }}; font-size: {{ block.settings.font_size }}px;"><span style="color: {{ block.settings.icon_color }};">{{ block.settings.icon }}</span> {{ block.settings.bullet_4 }}</li>{%- endif -%}
          {%- if block.settings.bullet_5 != blank -%}<li style="color: {{ block.settings.text_color }}; font-size: {{ block.settings.font_size }}px;"><span style="color: {{ block.settings.icon_color }};">{{ block.settings.icon }}</span> {{ block.settings.bullet_5 }}</li>{%- endif -%}
          {%- if block.settings.bullet_6 != blank -%}<li style="color: {{ block.settings.text_color }}; font-size: {{ block.settings.font_size }}px;"><span style="color: {{ block.settings.icon_color }};">{{ block.settings.icon }}</span> {{ block.settings.bullet_6 }}</li>{%- endif -%}
        </ul>

        {# BENEFIT PILLS #}
        {%- elsif block.type == 'benefit_pills' -%}
        <div class="cp-pills" style="margin-bottom: {{ block.settings.margin_bottom }}px; gap: {{ block.settings.gap }}px;" {{ block.shopify_attributes }}>
          {%- if block.settings.pill_1 != blank -%}<span class="cp-pill" style="background: {{ block.settings.bg }}; color: {{ block.settings.text_color }}; border: 1px solid {{ block.settings.border_color }}; border-radius: {{ block.settings.border_radius }}px; font-size: {{ block.settings.font_size }}px; padding: {{ block.settings.pad_v }}px {{ block.settings.pad_h }}px;">{{ block.settings.icon_1 }} {{ block.settings.pill_1 }}</span>{%- endif -%}
          {%- if block.settings.pill_2 != blank -%}<span class="cp-pill" style="background: {{ block.settings.bg }}; color: {{ block.settings.text_color }}; border: 1px solid {{ block.settings.border_color }}; border-radius: {{ block.settings.border_radius }}px; font-size: {{ block.settings.font_size }}px; padding: {{ block.settings.pad_v }}px {{ block.settings.pad_h }}px;">{{ block.settings.icon_2 }} {{ block.settings.pill_2 }}</span>{%- endif -%}
          {%- if block.settings.pill_3 != blank -%}<span class="cp-pill" style="background: {{ block.settings.bg }}; color: {{ block.settings.text_color }}; border: 1px solid {{ block.settings.border_color }}; border-radius: {{ block.settings.border_radius }}px; font-size: {{ block.settings.font_size }}px; padding: {{ block.settings.pad_v }}px {{ block.settings.pad_h }}px;">{{ block.settings.icon_3 }} {{ block.settings.pill_3 }}</span>{%- endif -%}
          {%- if block.settings.pill_4 != blank -%}<span class="cp-pill" style="background: {{ block.settings.bg }}; color: {{ block.settings.text_color }}; border: 1px solid {{ block.settings.border_color }}; border-radius: {{ block.settings.border_radius }}px; font-size: {{ block.settings.font_size }}px; padding: {{ block.settings.pad_v }}px {{ block.settings.pad_h }}px;">{{ block.settings.icon_4 }} {{ block.settings.pill_4 }}</span>{%- endif -%}
        </div>

        {# PRICE #}
        {%- elsif block.type == 'price' -%}
        <div class="cp-price" style="margin-bottom: {{ block.settings.margin_bottom }}px;" {{ block.shopify_attributes }}>
          <span class="cp-price-current" style="color: {{ block.settings.price_color }}; font-size: {{ block.settings.price_size }}px; font-weight: {{ block.settings.price_weight }};">
            {{ current_variant.price | money }}
          </span>
          {%- if current_variant.compare_at_price > current_variant.price -%}
          <span class="cp-price-compare" style="color: {{ block.settings.compare_color }}; font-size: {{ block.settings.compare_size }}px; text-decoration: line-through; margin-left: 8px;">
            {{ current_variant.compare_at_price | money }}
          </span>
          {%- if block.settings.show_badge -%}
          {%- assign save_pct = current_variant.compare_at_price | minus: current_variant.price | times: 100 | divided_by: current_variant.compare_at_price -%}
          <span class="cp-save-badge" style="background: {{ block.settings.badge_bg }}; color: {{ block.settings.badge_color }}; font-size: 11px; font-weight: 700; padding: 2px 8px; border-radius: 4px; margin-left: 8px;">
            {{ block.settings.badge_prefix }}{{ save_pct }}%
          </span>
          {%- endif -%}
          {%- endif -%}
        </div>

        {# SHIPPING NOTICE #}
        {%- elsif block.type == 'shipping_notice' -%}
        <div class="cp-shipping" style="background: {{ block.settings.bg }}; border: 1px solid {{ block.settings.border_color }}; border-radius: {{ block.settings.border_radius }}px; padding: {{ block.settings.pad_v }}px {{ block.settings.pad_h }}px; color: {{ block.settings.text_color }}; font-size: {{ block.settings.font_size }}px; margin-bottom: {{ block.settings.margin_bottom }}px; display: flex; align-items: center; gap: 8px;" {{ block.shopify_attributes }}>
          <span style="color: {{ block.settings.dot_color }}; font-size: 10px;">●</span>
          {{ block.settings.text }}
        </div>

        {# ABOVE ATC TEXT #}
        {%- elsif block.type == 'above_atc' -%}
        {%- if block.settings.text != blank -%}
        <div class="cp-above-atc" style="text-align: center; color: {{ block.settings.text_color }}; font-size: {{ block.settings.font_size }}px; background: {{ block.settings.bg }}; border-radius: {{ block.settings.border_radius }}px; padding: {{ block.settings.pad_v }}px {{ block.settings.pad_h }}px; margin-bottom: {{ block.settings.margin_bottom }}px;" {{ block.shopify_attributes }}>
          {{ block.settings.text }}
        </div>
        {%- endif -%}

        {# ADD TO CART #}
        {%- elsif block.type == 'add_to_cart' -%}
        {%- form 'product', cp, id: 'custom-product-form' -%}
          <input type="hidden" name="id" value="{{ current_variant.id }}">
          <button
            type="submit"
            name="add"
            class="cp-atc"
            style="
              background: {{ block.settings.bg }};
              color: {{ block.settings.text_color }};
              border: {{ block.settings.border_width }}px solid {{ block.settings.border_color }};
              border-radius: {{ block.settings.border_radius }}px;
              font-size: {{ block.settings.font_size }}px;
              font-weight: {{ block.settings.font_weight }};
              padding: {{ block.settings.pad_v }}px {{ block.settings.pad_h }}px;
              width: 100%;
              cursor: pointer;
              letter-spacing: {{ block.settings.letter_spacing }}px;
              text-transform: {{ block.settings.text_transform }};
              margin-bottom: {{ block.settings.margin_bottom }}px;
              display: flex;
              align-items: center;
              justify-content: center;
              gap: 8px;
            "
          >
            {%- if block.settings.show_icon -%}
            <svg width="{{ block.settings.icon_size }}" height="{{ block.settings.icon_size }}" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><circle cx="9" cy="21" r="1"/><circle cx="20" cy="21" r="1"/><path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"/></svg>
            {%- endif -%}
            {{ block.settings.text }}
            {%- if block.settings.show_price -%}
            <span style="opacity: 0.9; font-weight: 500;">— {{ current_variant.price | money }}</span>
            {%- endif -%}
          </button>
        {%- endform -%}

        {# DYNAMIC CHECKOUT (PayPal etc) #}
        {%- elsif block.type == 'dynamic_checkout' -%}
        {%- if block.settings.show -%}
        <div class="cp-dynamic-checkout" style="margin-bottom: {{ block.settings.margin_bottom }}px;" {{ block.shopify_attributes }}>
          {{ form | payment_button }}
        </div>
        {%- endif -%}

        {# PAYMENT ICONS #}
        {%- elsif block.type == 'payment_icons' -%}
        <div class="cp-payment" style="display: flex; flex-wrap: wrap; justify-content: {{ block.settings.align }}; gap: {{ block.settings.gap }}px; margin-bottom: {{ block.settings.margin_bottom }}px; margin-top: {{ block.settings.margin_top }}px;" {{ block.shopify_attributes }}>
          {%- for type in shop.enabled_payment_types -%}
          {{ type | payment_type_svg_tag: style: 'height: ' | append: block.settings.icon_height | append: 'px; opacity: 0.75;' }}
          {%- endfor -%}
        </div>

        {# TRUST BADGES #}
        {%- elsif block.type == 'trust_badges' -%}
        <div class="cp-trust" style="display: flex; justify-content: {{ block.settings.align }}; gap: {{ block.settings.gap }}px; flex-wrap: wrap; border-top: 1px solid {{ block.settings.divider_color }}; padding-top: {{ block.settings.pad_top }}px; margin-bottom: {{ block.settings.margin_bottom }}px;" {{ block.shopify_attributes }}>
          {%- if block.settings.badge_1_title != blank -%}
          <div style="text-align: center; display: flex; flex-direction: column; align-items: center; gap: 3px; max-width: {{ block.settings.badge_width }}px;">
            <div style="font-size: {{ block.settings.icon_size }}px;">{{ block.settings.badge_1_icon }}</div>
            <div style="font-size: {{ block.settings.title_size }}px; font-weight: 600; color: {{ block.settings.title_color }}; line-height: 1.2;">{{ block.settings.badge_1_title }}</div>
            <div style="font-size: {{ block.settings.sub_size }}px; color: {{ block.settings.sub_color }}; line-height: 1.2;">{{ block.settings.badge_1_sub }}</div>
          </div>
          {%- endif -%}
          {%- if block.settings.badge_2_title != blank -%}
          <div style="text-align: center; display: flex; flex-direction: column; align-items: center; gap: 3px; max-width: {{ block.settings.badge_width }}px;">
            <div style="font-size: {{ block.settings.icon_size }}px;">{{ block.settings.badge_2_icon }}</div>
            <div style="font-size: {{ block.settings.title_size }}px; font-weight: 600; color: {{ block.settings.title_color }}; line-height: 1.2;">{{ block.settings.badge_2_title }}</div>
            <div style="font-size: {{ block.settings.sub_size }}px; color: {{ block.settings.sub_color }}; line-height: 1.2;">{{ block.settings.badge_2_sub }}</div>
          </div>
          {%- endif -%}
          {%- if block.settings.badge_3_title != blank -%}
          <div style="text-align: center; display: flex; flex-direction: column; align-items: center; gap: 3px; max-width: {{ block.settings.badge_width }}px;">
            <div style="font-size: {{ block.settings.icon_size }}px;">{{ block.settings.badge_3_icon }}</div>
            <div style="font-size: {{ block.settings.title_size }}px; font-weight: 600; color: {{ block.settings.title_color }}; line-height: 1.2;">{{ block.settings.badge_3_title }}</div>
            <div style="font-size: {{ block.settings.sub_size }}px; color: {{ block.settings.sub_color }}; line-height: 1.2;">{{ block.settings.badge_3_sub }}</div>
          </div>
          {%- endif -%}
        </div>

        {# GUARANTEE BADGE #}
        {%- elsif block.type == 'guarantee' -%}
        <div class="cp-guarantee" style="background: {{ block.settings.bg }}; border: 1px solid {{ block.settings.border_color }}; border-radius: {{ block.settings.border_radius }}px; padding: {{ block.settings.pad_v }}px {{ block.settings.pad_h }}px; display: flex; align-items: center; gap: 12px; margin-bottom: {{ block.settings.margin_bottom }}px;" {{ block.shopify_attributes }}>
          <div style="font-size: {{ block.settings.icon_size }}px; flex-shrink: 0; line-height: 1;">{{ block.settings.icon }}</div>
          <div>
            <div style="font-size: {{ block.settings.title_size }}px; font-weight: 700; color: {{ block.settings.title_color }}; margin-bottom: 3px; line-height: 1.2;">{{ block.settings.title }}</div>
            <div style="font-size: {{ block.settings.text_size }}px; color: {{ block.settings.text_color }}; line-height: 1.4;">{{ block.settings.text }}</div>
          </div>
        </div>

        {# FEATURED REVIEW #}
        {%- elsif block.type == 'featured_review' -%}
        <div class="cp-review" style="background: {{ block.settings.bg }}; border: 1px solid {{ block.settings.border_color }}; border-radius: {{ block.settings.border_radius }}px; padding: {{ block.settings.pad_v }}px {{ block.settings.pad_h }}px; margin-bottom: {{ block.settings.margin_bottom }}px;" {{ block.shopify_attributes }}>
          <div style="color: {{ block.settings.star_color }}; font-size: {{ block.settings.star_size }}px; margin-bottom: 6px;">★★★★★</div>
          <div style="font-size: {{ block.settings.quote_size }}px; font-style: italic; color: {{ block.settings.quote_color }}; margin-bottom: 8px; line-height: 1.5;">"{{ block.settings.quote }}"</div>
          <div style="font-size: {{ block.settings.name_size }}px; color: {{ block.settings.name_color }}; font-weight: 600;">— {{ block.settings.name }} · <span style="color: {{ block.settings.verified_color }};">✓ {{ block.settings.verified_label }}</span></div>
        </div>

        {# FAQ ACCORDION #}
        {%- elsif block.type == 'faq' -%}
        <div class="cp-faq" style="margin-bottom: {{ block.settings.margin_bottom }}px;" {{ block.shopify_attributes }}>
          {%- if block.settings.q1 != blank -%}
          <details class="cp-faq-item" style="border-bottom: 1px solid {{ block.settings.border_color }};">
            <summary class="cp-faq-q" style="color: {{ block.settings.q_color }}; font-size: {{ block.settings.q_size }}px; font-weight: {{ block.settings.q_weight }}; padding: {{ block.settings.item_pad }}px 0; cursor: pointer; list-style: none; display: flex; justify-content: space-between;">{{ block.settings.q1 }}<span class="cp-faq-icon">+</span></summary>
            <div class="cp-faq-a" style="color: {{ block.settings.a_color }}; font-size: {{ block.settings.a_size }}px; padding-bottom: {{ block.settings.item_pad }}px; line-height: 1.6;">{{ block.settings.a1 }}</div>
          </details>
          {%- endif -%}
          {%- if block.settings.q2 != blank -%}
          <details class="cp-faq-item" style="border-bottom: 1px solid {{ block.settings.border_color }};">
            <summary class="cp-faq-q" style="color: {{ block.settings.q_color }}; font-size: {{ block.settings.q_size }}px; font-weight: {{ block.settings.q_weight }}; padding: {{ block.settings.item_pad }}px 0; cursor: pointer; list-style: none; display: flex; justify-content: space-between;">{{ block.settings.q2 }}<span class="cp-faq-icon">+</span></summary>
            <div class="cp-faq-a" style="color: {{ block.settings.a_color }}; font-size: {{ block.settings.a_size }}px; padding-bottom: {{ block.settings.item_pad }}px; line-height: 1.6;">{{ block.settings.a2 }}</div>
          </details>
          {%- endif -%}
          {%- if block.settings.q3 != blank -%}
          <details class="cp-faq-item" style="border-bottom: 1px solid {{ block.settings.border_color }};">
            <summary class="cp-faq-q" style="color: {{ block.settings.q_color }}; font-size: {{ block.settings.q_size }}px; font-weight: {{ block.settings.q_weight }}; padding: {{ block.settings.item_pad }}px 0; cursor: pointer; list-style: none; display: flex; justify-content: space-between;">{{ block.settings.q3 }}<span class="cp-faq-icon">+</span></summary>
            <div class="cp-faq-a" style="color: {{ block.settings.a_color }}; font-size: {{ block.settings.a_size }}px; padding-bottom: {{ block.settings.item_pad }}px; line-height: 1.6;">{{ block.settings.a3 }}</div>
          </details>
          {%- endif -%}
          {%- if block.settings.q4 != blank -%}
          <details class="cp-faq-item" style="border-bottom: 1px solid {{ block.settings.border_color }};">
            <summary class="cp-faq-q" style="color: {{ block.settings.q_color }}; font-size: {{ block.settings.q_size }}px; font-weight: {{ block.settings.q_weight }}; padding: {{ block.settings.item_pad }}px 0; cursor: pointer; list-style: none; display: flex; justify-content: space-between;">{{ block.settings.q4 }}<span class="cp-faq-icon">+</span></summary>
            <div class="cp-faq-a" style="color: {{ block.settings.a_color }}; font-size: {{ block.settings.a_size }}px; padding-bottom: {{ block.settings.item_pad }}px; line-height: 1.6;">{{ block.settings.a4 }}</div>
          </details>
          {%- endif -%}
          {%- if block.settings.q5 != blank -%}
          <details class="cp-faq-item" style="border-bottom: 1px solid {{ block.settings.border_color }};">
            <summary class="cp-faq-q" style="color: {{ block.settings.q_color }}; font-size: {{ block.settings.q_size }}px; font-weight: {{ block.settings.q_weight }}; padding: {{ block.settings.item_pad }}px 0; cursor: pointer; list-style: none; display: flex; justify-content: space-between;">{{ block.settings.q5 }}<span class="cp-faq-icon">+</span></summary>
            <div class="cp-faq-a" style="color: {{ block.settings.a_color }}; font-size: {{ block.settings.a_size }}px; padding-bottom: {{ block.settings.item_pad }}px; line-height: 1.6;">{{ block.settings.a5 }}</div>
          </details>
          {%- endif -%}
        </div>

        {# DIVIDER #}
        {%- elsif block.type == 'divider' -%}
        <hr style="border: none; border-top: {{ block.settings.width }}px solid {{ block.settings.color }}; margin: {{ block.settings.margin }}px 0;" {{ block.shopify_attributes }}>

        {# CUSTOM TEXT #}
        {%- elsif block.type == 'custom_text' -%}
        <div style="color: {{ block.settings.color }}; font-size: {{ block.settings.font_size }}px; line-height: {{ block.settings.line_height }}; margin-bottom: {{ block.settings.margin_bottom }}px; text-align: {{ block.settings.align }};" {{ block.shopify_attributes }}>
          {{ block.settings.text }}
        </div>

      {%- endif -%}
      {%- endfor -%}

    </div>
  </div>
</div>

<style>
.cp-wrapper { width: 100%; }
.cp-container {
  max-width: {{ section.settings.container_width }}px;
  margin: 0 auto;
  padding: {{ section.settings.pad_top }}px {{ section.settings.pad_h }}px {{ section.settings.pad_bottom }}px;
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start;
  gap: {{ section.settings.column_gap }}px;
}
/* Gallery column — 55% like Elixir (flex: 0 0 55%) */
.cp-gallery {
  flex: 0 0 {{ section.settings.gallery_percent }}%;
  max-width: {{ section.settings.gallery_percent }}%;
  /* NO position:sticky — causes gallery to follow user while scrolling */
  position: static;
  overflow: hidden;
  order: 1; /* gallery on the LEFT — matches competitor layout */
}
.cp-buybox {
  flex: 0 0 calc({{ section.settings.buybox_percent }}% - {{ section.settings.column_gap }}px);
  max-width: calc({{ section.settings.buybox_percent }}% - {{ section.settings.column_gap }}px);
  order: 2; /* buy box on the RIGHT */
}
.cp-main-image {
  position: relative;
}
/* Buy box column — remaining width */
.cp-buybox {
  flex: 0 0 calc({{ section.settings.buybox_percent }}% - {{ section.settings.column_gap }}px);
  max-width: calc({{ section.settings.buybox_percent }}% - {{ section.settings.column_gap }}px);
}
.cp-main-image { width: 100%; position: relative; overflow: hidden; }
.cp-main-image img {
  width: 100%;
  height: auto;
  display: block;
  object-fit: cover;
  aspect-ratio: {{ section.settings.image_aspect_ratio }};
  border-radius: {{ section.settings.image_border_radius }}px;
}
.cp-thumbnails {
  display: grid;
  grid-template-columns: repeat({{ section.settings.thumb_columns }}, 1fr);
  gap: {{ section.settings.thumb_gap }}px;
  margin-top: {{ section.settings.thumb_gap }}px;
}
.cp-thumb { border: 2px solid transparent; background: none; padding: 0; cursor: pointer; border-radius: {{ section.settings.image_border_radius }}px; overflow: hidden; }
.cp-thumb--active { border-color: {{ section.settings.thumb_active_color }}; }
.cp-thumb img { width: 100%; height: auto; display: block; }
.cp-buybox { }
.cp-rating { display: flex; align-items: center; gap: 8px; }
.cp-pills { display: flex; flex-wrap: wrap; }
.cp-bullets { list-style: none; padding: 0; margin: 0; display: flex; flex-direction: column; }
.cp-bullets li { display: flex; align-items: flex-start; gap: 8px; }
.cp-atc { transition: opacity 0.2s; }
.cp-atc:hover { opacity: 0.88; }
.cp-faq-item[open] .cp-faq-icon { content: '−'; transform: rotate(45deg); }
.cp-faq-icon { transition: transform 0.2s; flex-shrink: 0; }
details > summary::-webkit-details-marker { display: none; }
/* ─── MOBILE (base — no query needed, flex-direction already set) ─── */
/* The container defaults to column on mobile via JS or we override: */
@media (max-width: 749px) {
  .cp-container {
    flex-direction: column !important;
    gap: 20px;
    padding: 16px 16px 24px;
  }
  .cp-gallery,
  .cp-buybox {
    flex: 0 0 100% !important;
    max-width: 100% !important;
    position: static !important;
    width: 100% !important;
  }
  /* Mobile: show thumbnails as horizontal scroll */
  .cp-thumbnails {
    display: flex !important;
    flex-direction: row;
    overflow-x: auto;
    gap: 8px;
    grid-template-columns: unset;
    -webkit-overflow-scrolling: touch;
    scrollbar-width: none;
  }
  .cp-thumbnails::-webkit-scrollbar { display: none; }
  .cp-thumb {
    display: block !important;
    flex: 0 0 72px;
    width: 72px;
    height: 72px;
  }
  /* Mobile buy box adjustments */
  .cp-rating { flex-wrap: wrap; gap: 4px; }
  .cp-pills { gap: 6px; }
  .cp-atc {
    min-height: 52px !important;  /* touch target */
    font-size: 16px !important;
    width: 100% !important;
  }
  .cp-trust { gap: 12px; }
  .cp-title { font-size: 22px; line-height: 1.2; }
}
</style>

<script>
document.addEventListener('DOMContentLoaded', function() {
  var thumbs = document.querySelectorAll('.cp-thumb');
  var mainImg = document.getElementById('cp-main-img');
  if (!mainImg) return;
  thumbs.forEach(function(thumb) {
    thumb.addEventListener('click', function() {
      mainImg.src = this.dataset.src;
      thumbs.forEach(function(t) { t.classList.remove('cp-thumb--active'); });
      this.classList.add('cp-thumb--active');
    });
  });
});
</script>

{% schema %}
{
  "name": "Custom Product",
  "settings": [
    { "type": "header", "content": "Layout" },
    { "type": "color", "id": "page_bg", "label": "Page background", "default": "#ffffff" },
    { "type": "range", "id": "container_width", "label": "Container max width", "min": 900, "max": 1600, "step": 20, "default": 1200 },
    { "type": "range", "id": "pad_top", "label": "Padding top", "min": 0, "max": 80, "step": 4, "default": 40 },
    { "type": "range", "id": "pad_bottom", "label": "Padding bottom", "min": 0, "max": 80, "step": 4, "default": 40 },
    { "type": "range", "id": "pad_h", "label": "Padding horizontal", "min": 0, "max": 60, "step": 4, "default": 20 },
    { "type": "range", "id": "column_gap",       "label": "Gap between columns (px)", "min": 16, "max": 80, "step": 4, "default": 40 },
    { "type": "range", "id": "gallery_percent",  "label": "Gallery column width (%)", "min": 40, "max": 65, "step": 1, "default": 55, "info": "Elixir standard: 55%. Remaining goes to buy box." },
    { "type": "range", "id": "buybox_percent",   "label": "Buy box column width (%)", "min": 35, "max": 60, "step": 1, "default": 45 },
    { "type": "header", "content": "Gallery" },
    { "type": "range",  "id": "image_border_radius", "label": "Image border radius", "min": 0, "max": 24, "step": 2, "default": 8 },
    { "type": "select", "id": "image_aspect_ratio",  "label": "Image aspect ratio",
      "options": [{"value":"1/1","label":"Square (1:1)"},{"value":"4/5","label":"Portrait (4:5)"},{"value":"3/4","label":"Portrait (3:4)"},{"value":"auto","label":"Auto (natural)"}],
      "default": "1/1" },
    { "type": "range", "id": "thumb_columns", "label": "Thumbnail columns", "min": 3, "max": 8, "step": 1, "default": 5 },
    { "type": "range", "id": "thumb_gap", "label": "Thumbnail gap", "min": 4, "max": 16, "step": 2, "default": 8 },
    { "type": "color", "id": "thumb_active_color", "label": "Active thumbnail border", "default": "#111111" }
  ],
  "blocks": [
    {
      "type": "rating", "name": "Star Rating",
      "settings": [
        { "type": "text", "id": "text", "label": "Rating text", "default": "4.9 · Excellent · 1,407 reviews" },
        { "type": "color", "id": "star_color", "label": "Star color", "default": "#f59e0b" },
        { "type": "range", "id": "star_size", "label": "Star size", "min": 12, "max": 28, "step": 1, "default": 18 },
        { "type": "color", "id": "text_color", "label": "Text color", "default": "#374151" },
        { "type": "range", "id": "font_size", "label": "Font size", "min": 11, "max": 16, "step": 1, "default": 13 },
        { "type": "select", "id": "font_weight", "label": "Font weight", "options": [{"value":"400","label":"Normal"},{"value":"600","label":"Semibold"},{"value":"700","label":"Bold"}], "default": "400" },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 24, "step": 2, "default": 4 }
      ]
    },
    {
      "type": "social_proof", "name": "Social Proof Line",
      "settings": [
        { "type": "text", "id": "text", "label": "Text", "default": "El 89% de los clientes:" },
        { "type": "color", "id": "color", "label": "Color", "default": "#555555" },
        { "type": "range", "id": "font_size", "label": "Font size", "min": 11, "max": 16, "step": 1, "default": 13 },
        { "type": "select", "id": "font_weight", "label": "Weight", "options": [{"value":"400","label":"Normal"},{"value":"600","label":"Semibold"},{"value":"700","label":"Bold"}], "default": "400" },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 24, "step": 2, "default": 4 }
      ]
    },
    {
      "type": "title", "name": "Product Title",
      "settings": [
        { "type": "color", "id": "color", "label": "Color", "default": "#111111" },
        { "type": "range", "id": "font_size", "label": "Font size", "min": 20, "max": 56, "step": 2, "default": 32 },
        { "type": "text", "id": "line_height", "label": "Line height", "default": "1.1" },
        { "type": "select", "id": "font_weight", "label": "Weight", "options": [{"value":"600","label":"Semibold"},{"value":"700","label":"Bold"},{"value":"800","label":"Extrabold"}], "default": "700" },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 24, "step": 2, "default": 8 }
      ]
    },
    {
      "type": "subtitle", "name": "Subtitle / Tagline",
      "settings": [
        { "type": "text", "id": "text", "label": "Text" },
        { "type": "color", "id": "color", "label": "Color", "default": "#6b7280" },
        { "type": "range", "id": "font_size", "label": "Font size", "min": 11, "max": 18, "step": 1, "default": 13 },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 24, "step": 2, "default": 8 }
      ]
    },
    {
      "type": "benefit_bullets", "name": "Benefit Bullets",
      "settings": [
        { "type": "text", "id": "icon", "label": "Icon (emoji or symbol)", "default": "✓" },
        { "type": "color", "id": "icon_color", "label": "Icon color", "default": "#c0392b" },
        { "type": "color", "id": "text_color", "label": "Text color", "default": "#374151" },
        { "type": "range", "id": "font_size", "label": "Font size", "min": 12, "max": 18, "step": 1, "default": 14 },
        { "type": "range", "id": "gap", "label": "Gap between bullets", "min": 4, "max": 16, "step": 2, "default": 8 },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 24, "step": 2, "default": 12 },
        { "type": "text", "id": "bullet_1", "label": "Bullet 1" },
        { "type": "text", "id": "bullet_2", "label": "Bullet 2" },
        { "type": "text", "id": "bullet_3", "label": "Bullet 3" },
        { "type": "text", "id": "bullet_4", "label": "Bullet 4" },
        { "type": "text", "id": "bullet_5", "label": "Bullet 5" },
        { "type": "text", "id": "bullet_6", "label": "Bullet 6" }
      ]
    },
    {
      "type": "benefit_pills", "name": "Benefit Pills",
      "settings": [
        { "type": "color", "id": "bg", "label": "Background", "default": "#fef2f2" },
        { "type": "color", "id": "text_color", "label": "Text color", "default": "#991b1b" },
        { "type": "color", "id": "border_color", "label": "Border color", "default": "#fecaca" },
        { "type": "range", "id": "border_radius", "label": "Border radius", "min": 0, "max": 50, "step": 2, "default": 20 },
        { "type": "range", "id": "font_size", "label": "Font size", "min": 10, "max": 15, "step": 1, "default": 12 },
        { "type": "range", "id": "pad_v", "label": "Padding vertical", "min": 2, "max": 12, "step": 1, "default": 5 },
        { "type": "range", "id": "pad_h", "label": "Padding horizontal", "min": 6, "max": 20, "step": 2, "default": 10 },
        { "type": "range", "id": "gap", "label": "Gap", "min": 4, "max": 16, "step": 2, "default": 8 },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 24, "step": 2, "default": 12 },
        { "type": "text", "id": "icon_1", "label": "Icon 1 (emoji)" }, { "type": "text", "id": "pill_1", "label": "Pill 1" },
        { "type": "text", "id": "icon_2", "label": "Icon 2 (emoji)" }, { "type": "text", "id": "pill_2", "label": "Pill 2" },
        { "type": "text", "id": "icon_3", "label": "Icon 3 (emoji)" }, { "type": "text", "id": "pill_3", "label": "Pill 3" },
        { "type": "text", "id": "icon_4", "label": "Icon 4 (emoji)" }, { "type": "text", "id": "pill_4", "label": "Pill 4" }
      ]
    },
    {
      "type": "price", "name": "Price",
      "settings": [
        { "type": "color", "id": "price_color", "label": "Price color", "default": "#111111" },
        { "type": "range", "id": "price_size", "label": "Price size", "min": 18, "max": 40, "step": 1, "default": 26 },
        { "type": "select", "id": "price_weight", "label": "Price weight", "options": [{"value":"400","label":"Normal"},{"value":"600","label":"Semibold"},{"value":"700","label":"Bold"}], "default": "700" },
        { "type": "color", "id": "compare_color", "label": "Compare price color", "default": "#9ca3af" },
        { "type": "range", "id": "compare_size", "label": "Compare price size", "min": 14, "max": 28, "step": 1, "default": 18 },
        { "type": "checkbox", "id": "show_badge", "label": "Show save % badge", "default": true },
        { "type": "color", "id": "badge_bg", "label": "Badge background", "default": "#dc2626" },
        { "type": "color", "id": "badge_color", "label": "Badge text color", "default": "#ffffff" },
        { "type": "text", "id": "badge_prefix", "label": "Badge prefix", "default": "Ahorra " },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 24, "step": 2, "default": 12 }
      ]
    },
    {
      "type": "shipping_notice", "name": "Shipping Notice",
      "settings": [
        { "type": "text", "id": "text", "label": "Text", "default": "Listo para enviar. Llega aprox. en 5–7 días hábiles." },
        { "type": "color", "id": "bg", "label": "Background", "default": "#f9fafb" },
        { "type": "color", "id": "border_color", "label": "Border color", "default": "#e5e7eb" },
        { "type": "range", "id": "border_radius", "label": "Border radius", "min": 0, "max": 12, "step": 2, "default": 6 },
        { "type": "color", "id": "text_color", "label": "Text color", "default": "#374151" },
        { "type": "color", "id": "dot_color", "label": "Dot color", "default": "#22c55e" },
        { "type": "range", "id": "font_size", "label": "Font size", "min": 11, "max": 15, "step": 1, "default": 13 },
        { "type": "range", "id": "pad_v", "label": "Padding vertical", "min": 6, "max": 18, "step": 2, "default": 10 },
        { "type": "range", "id": "pad_h", "label": "Padding horizontal", "min": 8, "max": 24, "step": 2, "default": 14 },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 24, "step": 2, "default": 12 }
      ]
    },
    {
      "type": "above_atc", "name": "Above-ATC Strip",
      "settings": [
        { "type": "text", "id": "text", "label": "Text (blank to hide)", "default": "90 días para comprobarlo. O te devolvemos el dinero." },
        { "type": "color", "id": "text_color", "label": "Text color", "default": "#6b7280" },
        { "type": "range", "id": "font_size", "label": "Font size", "min": 11, "max": 15, "step": 1, "default": 13 },
        { "type": "color", "id": "bg", "label": "Background", "default": "#f9fafb" },
        { "type": "range", "id": "border_radius", "label": "Border radius", "min": 0, "max": 12, "step": 2, "default": 6 },
        { "type": "range", "id": "pad_v", "label": "Padding vertical", "min": 4, "max": 16, "step": 2, "default": 8 },
        { "type": "range", "id": "pad_h", "label": "Padding horizontal", "min": 8, "max": 24, "step": 2, "default": 14 },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 20, "step": 2, "default": 10 }
      ]
    },
    {
      "type": "add_to_cart", "name": "Add to Cart Button",
      "limit": 1,
      "settings": [
        { "type": "text", "id": "text", "label": "Button text", "default": "Agregar al carrito" },
        { "type": "checkbox", "id": "show_price", "label": "Show price in button", "default": false },
        { "type": "checkbox", "id": "show_icon", "label": "Show cart icon", "default": false },
        { "type": "range", "id": "icon_size", "label": "Icon size", "min": 14, "max": 24, "step": 1, "default": 18 },
        { "type": "color", "id": "bg", "label": "Background", "default": "#111111" },
        { "type": "color", "id": "text_color", "label": "Text color", "default": "#ffffff" },
        { "type": "color", "id": "border_color", "label": "Border color", "default": "#111111" },
        { "type": "range", "id": "border_width", "label": "Border width", "min": 0, "max": 4, "step": 1, "default": 0 },
        { "type": "range", "id": "border_radius", "label": "Border radius", "min": 0, "max": 50, "step": 2, "default": 8 },
        { "type": "range", "id": "font_size", "label": "Font size", "min": 14, "max": 24, "step": 1, "default": 16 },
        { "type": "select", "id": "font_weight", "label": "Font weight", "options": [{"value":"400","label":"Normal"},{"value":"600","label":"Semibold"},{"value":"700","label":"Bold"}], "default": "700" },
        { "type": "range", "id": "pad_v", "label": "Padding vertical", "min": 10, "max": 28, "step": 2, "default": 16 },
        { "type": "range", "id": "pad_h", "label": "Padding horizontal", "min": 16, "max": 60, "step": 4, "default": 24 },
        { "type": "range", "id": "letter_spacing", "label": "Letter spacing", "min": 0, "max": 4, "step": 0.5, "default": 0 },
        { "type": "select", "id": "text_transform", "label": "Text transform", "options": [{"value":"none","label":"None"},{"value":"uppercase","label":"Uppercase"},{"value":"capitalize","label":"Capitalize"}], "default": "none" },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 20, "step": 2, "default": 8 }
      ]
    },
    {
      "type": "dynamic_checkout", "name": "Dynamic Checkout (PayPal etc)",
      "limit": 1,
      "settings": [
        { "type": "checkbox", "id": "show", "label": "Show dynamic checkout", "default": true },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 20, "step": 2, "default": 8 }
      ]
    },
    {
      "type": "payment_icons", "name": "Payment Icons",
      "settings": [
        { "type": "select", "id": "align", "label": "Alignment", "options": [{"value":"flex-start","label":"Left"},{"value":"center","label":"Center"},{"value":"flex-end","label":"Right"}], "default": "center" },
        { "type": "range", "id": "icon_height", "label": "Icon height", "min": 16, "max": 36, "step": 2, "default": 24 },
        { "type": "range", "id": "gap", "label": "Gap", "min": 4, "max": 20, "step": 2, "default": 8 },
        { "type": "range", "id": "margin_top", "label": "Margin top", "min": 0, "max": 16, "step": 2, "default": 4 },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 20, "step": 2, "default": 12 }
      ]
    },
    {
      "type": "trust_badges", "name": "Trust Badges",
      "settings": [
        { "type": "select", "id": "align", "label": "Alignment", "options": [{"value":"flex-start","label":"Left"},{"value":"center","label":"Center"},{"value":"space-around","label":"Space around"}], "default": "center" },
        { "type": "range", "id": "gap", "label": "Gap", "min": 8, "max": 40, "step": 4, "default": 20 },
        { "type": "range", "id": "badge_width", "label": "Badge max width", "min": 60, "max": 140, "step": 10, "default": 90 },
        { "type": "range", "id": "icon_size", "label": "Icon size", "min": 16, "max": 40, "step": 2, "default": 24 },
        { "type": "color", "id": "title_color", "label": "Title color", "default": "#111111" },
        { "type": "range", "id": "title_size", "label": "Title font size", "min": 10, "max": 14, "step": 1, "default": 11 },
        { "type": "color", "id": "sub_color", "label": "Subtitle color", "default": "#6b7280" },
        { "type": "range", "id": "sub_size", "label": "Subtitle font size", "min": 9, "max": 13, "step": 1, "default": 10 },
        { "type": "color", "id": "divider_color", "label": "Divider color", "default": "#e5e7eb" },
        { "type": "range", "id": "pad_top", "label": "Padding top", "min": 8, "max": 24, "step": 2, "default": 12 },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 24, "step": 2, "default": 16 },
        { "type": "text", "id": "badge_1_icon", "label": "Badge 1 icon", "default": "🔄" },
        { "type": "text", "id": "badge_1_title", "label": "Badge 1 title", "default": "Devoluciones gratis" },
        { "type": "text", "id": "badge_1_sub", "label": "Badge 1 subtitle", "default": "Sin complicaciones" },
        { "type": "text", "id": "badge_2_icon", "label": "Badge 2 icon", "default": "🛡️" },
        { "type": "text", "id": "badge_2_title", "label": "Badge 2 title", "default": "90 días sin riesgo" },
        { "type": "text", "id": "badge_2_sub", "label": "Badge 2 subtitle", "default": "Garantía total" },
        { "type": "text", "id": "badge_3_icon", "label": "Badge 3 icon", "default": "🚚" },
        { "type": "text", "id": "badge_3_title", "label": "Badge 3 title", "default": "Envío gratis" },
        { "type": "text", "id": "badge_3_sub", "label": "Badge 3 subtitle", "default": "En todos los pedidos" }
      ]
    },
    {
      "type": "guarantee", "name": "Guarantee Badge",
      "settings": [
        { "type": "text", "id": "icon", "label": "Icon (emoji)", "default": "🛡️" },
        { "type": "range", "id": "icon_size", "label": "Icon size", "min": 24, "max": 64, "step": 4, "default": 40 },
        { "type": "text", "id": "title", "label": "Title", "default": "Garantía de devolución de 90 días" },
        { "type": "text", "id": "text", "label": "Description", "default": "Si no notas la diferencia, te devolvemos el 100%. Sin preguntas." },
        { "type": "color", "id": "bg", "label": "Background", "default": "#fffbeb" },
        { "type": "color", "id": "border_color", "label": "Border color", "default": "#fde68a" },
        { "type": "range", "id": "border_radius", "label": "Border radius", "min": 0, "max": 16, "step": 2, "default": 8 },
        { "type": "range", "id": "pad_v", "label": "Padding vertical", "min": 8, "max": 24, "step": 2, "default": 14 },
        { "type": "range", "id": "pad_h", "label": "Padding horizontal", "min": 12, "max": 30, "step": 2, "default": 16 },
        { "type": "color", "id": "title_color", "label": "Title color", "default": "#92400e" },
        { "type": "range", "id": "title_size", "label": "Title size", "min": 12, "max": 18, "step": 1, "default": 14 },
        { "type": "color", "id": "text_color", "label": "Text color", "default": "#78350f" },
        { "type": "range", "id": "text_size", "label": "Text size", "min": 11, "max": 15, "step": 1, "default": 13 },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 24, "step": 2, "default": 12 }
      ]
    },
    {
      "type": "featured_review", "name": "Featured Review",
      "settings": [
        { "type": "textarea", "id": "quote", "label": "Review quote" },
        { "type": "text", "id": "name", "label": "Reviewer name" },
        { "type": "text", "id": "verified_label", "label": "Verified label", "default": "Comprador verificado" },
        { "type": "color", "id": "star_color", "label": "Star color", "default": "#f59e0b" },
        { "type": "range", "id": "star_size", "label": "Star size", "min": 12, "max": 22, "step": 1, "default": 15 },
        { "type": "color", "id": "quote_color", "label": "Quote color", "default": "#374151" },
        { "type": "range", "id": "quote_size", "label": "Quote font size", "min": 12, "max": 16, "step": 1, "default": 14 },
        { "type": "color", "id": "name_color", "label": "Name color", "default": "#111111" },
        { "type": "range", "id": "name_size", "label": "Name font size", "min": 11, "max": 14, "step": 1, "default": 12 },
        { "type": "color", "id": "verified_color", "label": "Verified color", "default": "#16a34a" },
        { "type": "color", "id": "bg", "label": "Background", "default": "#f9fafb" },
        { "type": "color", "id": "border_color", "label": "Border color", "default": "#e5e7eb" },
        { "type": "range", "id": "border_radius", "label": "Border radius", "min": 0, "max": 16, "step": 2, "default": 8 },
        { "type": "range", "id": "pad_v", "label": "Padding vertical", "min": 8, "max": 24, "step": 2, "default": 14 },
        { "type": "range", "id": "pad_h", "label": "Padding horizontal", "min": 12, "max": 30, "step": 2, "default": 16 },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 24, "step": 2, "default": 12 }
      ]
    },
    {
      "type": "faq", "name": "FAQ Accordion",
      "settings": [
        { "type": "color", "id": "border_color", "label": "Divider color", "default": "#e5e7eb" },
        { "type": "color", "id": "q_color", "label": "Question color", "default": "#111111" },
        { "type": "range", "id": "q_size", "label": "Question font size", "min": 13, "max": 18, "step": 1, "default": 15 },
        { "type": "select", "id": "q_weight", "label": "Question weight", "options": [{"value":"400","label":"Normal"},{"value":"500","label":"Medium"},{"value":"600","label":"Semibold"}], "default": "500" },
        { "type": "color", "id": "a_color", "label": "Answer color", "default": "#6b7280" },
        { "type": "range", "id": "a_size", "label": "Answer font size", "min": 12, "max": 16, "step": 1, "default": 14 },
        { "type": "range", "id": "item_pad", "label": "Item padding", "min": 8, "max": 24, "step": 2, "default": 14 },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 30, "step": 2, "default": 16 },
        { "type": "text", "id": "q1", "label": "Q1" }, { "type": "textarea", "id": "a1", "label": "A1" },
        { "type": "text", "id": "q2", "label": "Q2" }, { "type": "textarea", "id": "a2", "label": "A2" },
        { "type": "text", "id": "q3", "label": "Q3" }, { "type": "textarea", "id": "a3", "label": "A3" },
        { "type": "text", "id": "q4", "label": "Q4" }, { "type": "textarea", "id": "a4", "label": "A4" },
        { "type": "text", "id": "q5", "label": "Q5" }, { "type": "textarea", "id": "a5", "label": "A5" }
      ]
    },
    { "type": "divider", "name": "Divider", "settings": [
        { "type": "color", "id": "color", "label": "Color", "default": "#e5e7eb" },
        { "type": "range", "id": "width", "label": "Width (px)", "min": 1, "max": 4, "step": 1, "default": 1 },
        { "type": "range", "id": "margin", "label": "Margin vertical", "min": 4, "max": 32, "step": 4, "default": 16 }
    ]},
    { "type": "custom_text", "name": "Custom Text", "settings": [
        { "type": "richtext", "id": "text", "label": "Text" },
        { "type": "color", "id": "color", "label": "Color", "default": "#374151" },
        { "type": "range", "id": "font_size", "label": "Font size", "min": 11, "max": 20, "step": 1, "default": 14 },
        { "type": "text", "id": "line_height", "label": "Line height", "default": "1.6" },
        { "type": "select", "id": "align", "label": "Alignment", "options": [{"value":"left","label":"Left"},{"value":"center","label":"Center"},{"value":"right","label":"Right"}], "default": "left" },
        { "type": "range", "id": "margin_bottom", "label": "Margin bottom", "min": 0, "max": 30, "step": 2, "default": 12 }
    ]}
  ],
  "presets": [{ "name": "Custom Product" }]
}
{% endschema %}
```

---

## Step 3 — Build `assets/custom-product.css`

Only structural CSS not already covered by inline styles:

```css
.cp-container { box-sizing: border-box; }
.cp-container * { box-sizing: border-box; }
.cp-bullets { list-style: none !important; padding: 0 !important; }
.cp-atc { border: none; font-family: inherit; transition: opacity 0.15s ease; }
.cp-atc:hover { opacity: 0.88; }
.cp-faq-item summary::-webkit-details-marker { display: none; }
.cp-faq-item[open] .cp-faq-icon { transform: rotate(45deg); }
.cp-faq-icon { transition: transform 0.2s ease; display: inline-block; }
```

---

## Step 4 — Update the product template JSON

**Replace `main-product` entirely.** The template `order` array must contain ONLY `custom-product`. Do NOT keep `main-product` alongside it — having both causes content to render twice and overlap.

If Shopify requires `main-product` for product page routing (rare), verify by testing without it first. Only add it back if the page 404s without it, and in that case add CSS to hide it completely: `display: none !important`.

```json
{
  "sections": {
    "custom_product_XXXXX": {
      "type": "custom-product",
      "blocks": {
        "rating_XXXXX":      { "type": "rating",         "settings": { "text": "RATING_TEXT", "star_color": "STAR_COLOR" } },
        "social_XXXXX":      { "type": "social_proof",   "settings": { "text": "SOCIAL_PROOF_LINE", "color": "COLOR_PRIMARY" } },
        "title_XXXXX":       { "type": "title",          "settings": { "color": "COLOR_TEXT", "font_size": 34 } },
        "subtitle_XXXXX":    { "type": "subtitle",       "settings": { "text": "PRODUCT_TAGLINE" } },
        "bullets_XXXXX":     { "type": "benefit_bullets","settings": { "icon": "✓", "icon_color": "COLOR_PRIMARY", "bullet_1": "BENEFIT_1", "bullet_2": "BENEFIT_2", "bullet_3": "BENEFIT_3", "bullet_4": "BENEFIT_4" } },
        "price_XXXXX":       { "type": "price",          "settings": { "badge_bg": "COLOR_PRIMARY" } },
        "shipping_XXXXX":    { "type": "shipping_notice","settings": { "text": "SHIPPING_TEXT" } },
        "above_atc_XXXXX":   { "type": "above_atc",      "settings": { "text": "GUARANTEE_LINE" } },
        "atc_XXXXX":         { "type": "add_to_cart",    "settings": { "text": "ATC_COPY", "bg": "COLOR_PRIMARY", "text_color": "#ffffff" } },
        "dynamic_XXXXX":     { "type": "dynamic_checkout","settings": { "show": true } },
        "payments_XXXXX":    { "type": "payment_icons",  "settings": {} },
        "badges_XXXXX":      { "type": "trust_badges",   "settings": { "badge_1_title": "BADGE_1", "badge_2_title": "BADGE_2", "badge_3_title": "BADGE_3" } },
        "guarantee_XXXXX":   { "type": "guarantee",      "settings": { "title": "GUARANTEE_TITLE", "text": "GUARANTEE_TEXT" } },
        "review_XXXXX":      { "type": "featured_review","settings": { "quote": "REVIEW_QUOTE", "name": "REVIEWER" } },
        "faq_XXXXX":         { "type": "faq",            "settings": { "q1": "FAQ_Q1", "a1": "FAQ_A1", "q2": "FAQ_Q2", "a2": "FAQ_A2", "q3": "FAQ_Q3", "a3": "FAQ_A3" } }
      },
      "block_order": ["rating_XXXXX","social_XXXXX","title_XXXXX","subtitle_XXXXX","bullets_XXXXX","price_XXXXX","shipping_XXXXX","above_atc_XXXXX","atc_XXXXX","dynamic_XXXXX","payments_XXXXX","badges_XXXXX","guarantee_XXXXX","review_XXXXX","faq_XXXXX"],
      "settings": { "page_bg": "#ffffff", "container_width": 1200, "column_gap": 48, "gallery_width": 1, "buybox_width": 1 }
    }
  },
  "order": ["custom_product_XXXXX"]
}
```

All CAPS placeholders are filled from competitor PDF analysis. Claude never asks the user for this content.

---

## Content generation rules

- **RATING_TEXT**: competitor's exact score and count in OUTPUT_LANGUAGE
- **SOCIAL_PROOF_LINE**: most powerful stat from competitor (e.g. "El 92% reportaron mejora en 3 semanas")
- **BENEFIT_1–6**: extracted from competitor's benefit bullets, in OUTPUT_LANGUAGE
- **ATC_COPY**: match competitor's CTA verb exactly in OUTPUT_LANGUAGE
- **BADGE_1–3**: match competitor's trust signals (guarantee days, returns, shipping)
- **FAQ_Q1–5 + answers**: extracted from competitor's buy box FAQ section
- **REVIEW_QUOTE**: most emotionally powerful quote from competitor's buy box

## Color contrast (mandatory)

Check every color pair before writing:
- Dark bg → white or very light text
- Light bg → dark text
- Never same-hue text on same-hue background
