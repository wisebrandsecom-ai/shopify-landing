# Template Builder — Reference

Loaded during Step 3 of the shopify-landing skill.
Instructions for creating a new product template in the working theme.

---

## How Shopify product templates work

A product template is a JSON file at `templates/product.TEMPLATE_NAME.json`.
It defines which sections appear on the product page and in what order.
Each section references a section type that exists in the theme's `sections/` folder.

The template file structure:
```json
{
  "sections": {
    "main": {
      "type": "main-product",
      "settings": { ... }
    },
    "section-id-1": {
      "type": "some-section-type",
      "settings": { ... }
    }
  },
  "order": ["main", "section-id-1", ...]
}
```

---

## Step 3.1 — Read the default product template

```bash
curl -s "https://STORE_URL/admin/api/2024-01/themes/WORKING_THEME_ID/assets.json\
?asset[key]=templates/product.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_TOKEN" | jq '.asset.value'
```

This gives you the base structure and available section types for this theme.

---

## Step 3.2 — List all available section types

```bash
curl -s "https://STORE_URL/admin/api/2024-01/themes/WORKING_THEME_ID/assets.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_TOKEN" | \
  jq '[.assets[].key | select(startswith("sections/"))]'
```

Map the competitor's `SECTION_STRUCTURE` to available section types in this theme:

| Competitor section | Look for in theme |
|-------------------|-------------------|
| Hero / main product | `main-product`, `product-hero` |
| Benefits grid | `multicolumn`, `features`, `benefits` |
| How it works | `steps`, `how-it-works`, `numbered-list` |
| Ingredients / specs | `collapsible-content`, `tabs`, `accordion` |
| Reviews / social proof | `product-reviews`, `testimonials`, `reviews` |
| FAQ | `faq`, `collapsible-content`, `accordion` |
| Trust badges | `trust-badges`, `icon-row`, `badges` |
| Rich text / story | `rich-text`, `featured-blog`, `content` |
| Before/after | `before-after`, `image-comparison` |
| Final CTA | `cta`, `banner`, `call-to-action` |

If the theme doesn't have a section type that matches → use the closest available alternative.
Do NOT invent section type names — only use what's confirmed to exist in `sections/`.

---

## Step 3.3 — Build the template JSON

Construct the template following the competitor's `SECTION_STRUCTURE` order.
Use only section types confirmed to exist in Step 3.2.

Example structure for a typical supplement product:
```json
{
  "sections": {
    "main": {
      "type": "main-product",
      "settings": {
        "show_vendor": false,
        "show_rating": true
      }
    },
    "benefits": {
      "type": "multicolumn",
      "settings": {
        "title": "Why PRODUCT_TITLE",
        "columns_desktop": 3
      }
    },
    "how-it-works": {
      "type": "rich-text",
      "settings": {
        "heading": "How It Works"
      }
    },
    "faq": {
      "type": "collapsible-content",
      "settings": {
        "heading": "Frequently Asked Questions"
      }
    }
  },
  "order": ["main", "benefits", "how-it-works", "faq"]
}
```

---

## Step 3.4 — Upload the template

```bash
curl -X PUT "https://STORE_URL/admin/api/2024-01/themes/WORKING_THEME_ID/assets.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "asset": {
      "key": "templates/product.TEMPLATE_NAME.json",
      "value": "<full JSON template content>"
    }
  }'
```

Verify:
```bash
curl -s "https://STORE_URL/admin/api/2024-01/themes/WORKING_THEME_ID/assets.json\
?asset[key]=templates/product.TEMPLATE_NAME.json" \
  -H "X-Shopify-Access-Token: $SHOPIFY_TOKEN" | jq '.asset.key'
```

Should return `"templates/product.TEMPLATE_NAME.json"`.

---

## Common issues

| Issue | Fix |
|-------|-----|
| Section type not found | Re-check sections list from Step 3.2, pick closest match |
| JSON parse error | Validate JSON before uploading — check all braces/quotes |
| Template not appearing in product editor | Wait 30 seconds and refresh — Shopify caches theme assets |
| `main-product` section missing | Every product template must include one `main-product` type section — never omit it |
