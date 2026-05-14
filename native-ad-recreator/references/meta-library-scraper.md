# Meta Ad Library Scraper — Reference

Uses Claude in Chrome (Claude_in_Chrome MCP) to extract static ads from a competitor's Meta Ad Library profile.

---

## Prerequisites

Claude in Chrome MCP must be connected. Verify with `screenshot` — if it returns a browser view, it's ready.

**Never use Gemini API, web_fetch, or any other method to access Meta Ad Library — only Claude in Chrome.**
Meta Ad Library blocks all non-browser requests. Claude in Chrome is the only method that works.

---

## Step M1 — Navigate to the Ad Library URL

```
navigate(url: AD_SOURCE)
```

Wait for page to load. Take a `screenshot` to confirm ads are visible.

If the page shows a login wall → navigate to:
`https://www.facebook.com/ads/library/?active_status=active&ad_type=all&country=ALL&q=BRAND_NAME&search_type=keyword_unordered`

---

## Step M2 — Scroll and collect static ads

Use `javascript_tool` to extract all visible ad cards:

```javascript
// Get all ad image URLs and copy text from the page
const ads = [];
const adCards = document.querySelectorAll('[data-testid="ad-card"], .x1i10hfl, ._4ik4, ._4ik5');

adCards.forEach((card, i) => {
  const img = card.querySelector('img');
  const body = card.querySelector('[data-ad-preview="message"], ._5pbx, .userContent');
  const headline = card.querySelector('._5s-i, .ellipsis, h6, ._5s-g');
  
  if (img && img.src && img.src.startsWith('http')) {
    ads.push({
      index: i + 1,
      image_url: img.src,
      copy: body ? body.innerText.trim() : '',
      headline: headline ? headline.innerText.trim() : ''
    });
  }
});

return JSON.stringify(ads);
```

If the selector doesn't work, take a `screenshot`, inspect what's visible, and adjust.

Scroll down with `javascript_tool`:
```javascript
window.scrollBy(0, 1000);
```
Repeat until no new ads load (typically 3-5 scrolls for 10-20 ads).

---

## Step M3 — Filter static ads only

From the extracted ads array, keep only static image ads:
- Include: ads with a single image (jpg/png/webp)
- Exclude: ads where the image URL contains `video`, `reel`, or has a play button overlay

Store as `ads[]` array: `[{ index, image_url, copy, headline }]`

Print count: `N static ads found`

---

## Step M4 — Download ad images locally

For each ad in `ads[]`:

```bash
curl -L "IMAGE_URL" -o "OUTPUT_FOLDER/anuncio_0N_reference.jpg"
```

These are reference images for visual analysis in Block 1.
They are NOT uploaded anywhere — analysis only.

---

## Step M5 — Store copy per ad

For each ad, store:
```
ad[n].copy_source = {
  primary_text: [body text from page],
  headline: [headline from page],
  description: [description if visible]
}
```

This feeds into Block 2 (copy adaptation).

---

## Notes

- Meta Ad Library is publicly accessible — no login required for basic browsing
- Image URLs from Meta CDN are direct and downloadable with curl
- If the JS selector fails, use `find` tool with keyword "Sponsored" or "Ad" to locate ad containers
- `read_page` can extract all text from the page if querySelector fails
- Always take a screenshot after navigation to confirm what's visible
