# Product Creator — GraphQL Reference

Loaded during Step 5 of the shopify-landing skill.
Contains the exact mutations to create the product with all required settings.

---

## Step 5.1 — Map product category to Shopify taxonomy

Use the competitor analysis `PRODUCT_CATEGORY` to find the correct Shopify product category ID.

Query the taxonomy:
```graphql
query {
  productCategories(first: 10, query: "PRODUCT_CATEGORY") {
    nodes { id name fullName }
  }
}
```

Pick the most specific match. Store as `TAXONOMY_ID`.

---

## Step 5.2 — Get all sales channel publication IDs

```graphql
query {
  publications(first: 10) {
    nodes { id name }
  }
}
```

Store all publication IDs as `PUBLICATION_IDS` list.

---

## Step 5.3 — Get location ID for inventory

```graphql
query {
  locations(first: 5) {
    nodes { id name }
  }
}
```

Store the primary location ID as `LOCATION_ID`.

---

## Step 5.4 — Create the product

```graphql
mutation productCreate($input: ProductInput!) {
  productCreate(input: $input) {
    product {
      id
      title
      handle
      status
      templateSuffix
      variants(first: 1) {
        nodes {
          id
          price
          compareAtPrice
          inventoryPolicy
          inventoryItem { id requiresShipping }
        }
      }
    }
    userErrors { field message }
  }
}
```

With these variables:
```json
{
  "input": {
    "title": "PRODUCT_TITLE",
    "status": "ACTIVE",
    "templateSuffix": "TEMPLATE_NAME",
    "productType": "PRODUCT_CATEGORY",
    "vendor": "LEGAL_ENTITY_NAME",
    "tags": ["auto-generated from key benefits and category"],
    "variants": [
      {
        "price": "PRODUCT_PRICE",
        "compareAtPrice": "COMPARE_AT_PRICE or omit if None",
        "inventoryPolicy": "DENY",
        "inventoryItem": {
          "tracked": false,
          "requiresShipping": true
        }
      }
    ],
    "category": { "id": "TAXONOMY_ID" }
  }
}
```

> ⚠️ Do NOT create bundle variants or multi-pack options. Bundling is handled by external apps (e.g. Kaching Bundles). Create a single default variant only.

---

## Step 5.5 — Publish to all sales channels

For each ID in `PUBLICATION_IDS`:
```graphql
mutation {
  publishablePublish(
    id: "PRODUCT_ID"
    input: { publicationId: "PUBLICATION_ID" }
  ) {
    publishable { ... on Product { id title } }
    userErrors { field message }
  }
}
```

---

## Step 5.6 — Auto-generate SEO fields

After creation, update the product's SEO:
```graphql
mutation {
  productUpdate(input: {
    id: "PRODUCT_ID"
    seo: {
      title: "PRODUCT_TITLE — [key benefit, ~5 words]"
      description: "[150-char description in product's language, benefit-focused]"
    }
  }) {
    product { id seo { title description } }
    userErrors { field message }
  }
}
```

---

## Error handling

| Error | Action |
|-------|--------|
| `Product title has already been taken` | Append a suffix or ask user for a different name |
| `Price must be a number` | Re-format price as string with 2 decimal places |
| `Invalid category` | Re-query taxonomy with broader search term |
| `Publication not found` | Skip that channel, continue with others |
| Any `userErrors` array non-empty | Log each error and attempt fix before continuing |
