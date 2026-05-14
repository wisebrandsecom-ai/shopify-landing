# Higgsfield Prompting — Reference

Used in **Block 3** (prompt construction) and **Block 4** (generation).

---

## Model selection

| Ad type | Model | When to use |
|---------|-------|-------------|
| Static standard | `nano_banana_2` | Product visible in ad — product shots, feature highlights, offer-based |
| Native / storytelling | `soul_2` | No product visible — UGC-style, lifestyle, person-led ads |

---

## Nano Banana 2 — Static ads

### Prompt structure

```
[SUBJECT] — [SETTING] — [COMPOSITION] — [STYLE] — [TEXT OVERLAYS] — [MOOD] — [TECHNICAL]
```

### Prompt template

```
{product_name} product shot. {visual_description_adapted}.
Setting: {setting}.
Composition: {composition_notes}.
Style: {style_reference} — clean, high-end commercial photography.
Color palette: {brand_colors}.
Mood: {mood}.
Text overlay: "{headline_text}" in bold {font_style} — {text_position}.
No logos. No watermarks. Photorealistic. Shot on medium format camera.
```

### Example (for reference)

```
Premium olive oil bottle product shot. Bottle placed on a rustic marble surface next to fresh herbs.
Setting: bright Mediterranean kitchen, natural window light.
Composition: centered product, rule of thirds, slight top-down angle.
Style: editorial food photography — clean, high-end commercial.
Color palette: warm whites, deep green, terracotta accents.
Mood: aspirational, wholesome, premium.
Text overlay: "Crafted for those who care" in bold sans-serif — bottom left corner.
No logos. No watermarks. Photorealistic. Shot on medium format camera.
```

### Nano Banana 2 — Do's and Don'ts

**Do:**
- Be very specific about the product placement and surface
- Specify lighting direction (natural, studio, golden hour, etc.)
- Describe text overlays with exact position (top-left, centered, bottom-right)
- Include color palette — it heavily influences output
- Mention "high-end commercial" or "editorial" to get clean results

**Don't:**
- Use vague terms like "beautiful" or "amazing" alone — pair with concrete descriptors
- Ask for multiple products in complex arrangements on first try — start simple
- Include brand logos (Higgsfield won't generate them accurately)
- Request very long text overlays — keep to under 6 words per overlay

---

## soul_2 — Native / storytelling ads

> **No Soul ID needed.** Generates realistic people from text description only.
> Use for UGC-style, lifestyle, person-led native ads where no product appears.

### Prompt structure

```
[CHARACTER ACTION] — [SETTING] — [EMOTIONAL BEAT] — [PRODUCT CONTEXT] — [STYLE]
```

### Prompt template

```
{character_description} in {setting}.
{action_or_gesture} while {product_context}.
Emotional tone: {mood} — feels like {style_reference}.
Natural lighting. Candid, authentic feel. UGC aesthetic.
{any_text_overlay_if_needed}
```

### Example (for reference)

```
Young woman in her early 30s sitting at a cozy home office desk.
Holding a coffee mug, looking relaxed and focused, laptop open with productivity app visible.
Emotional tone: calm, in-control, aspirational — feels like a real person's morning routine.
Natural window light. Candid, authentic feel. UGC aesthetic.
```

### Soul 1.0 — Do's and Don'ts

**Do:**
- Describe the character's action and emotional state concretely
- Reference the product naturally within the scene (not as the sole focus)
- Use "UGC aesthetic" or "candid feel" for native-looking output
- Keep text overlays minimal or absent — native ads rarely have heavy text

**Don't:**
- Use overly polished/commercial language — it breaks the native feel
- Ask for multiple people in complex interactions on first try
- Specify logos or branded elements on clothing

---

## Prompt construction logic (Block 3)

When building the image prompt, use this mapping:

| From Gemini extraction | Maps to prompt element |
|------------------------|------------------------|
| `visual_description` | Base composition and setting |
| `mood` | Mood and emotional tone |
| `color_palette` | Color palette (override with brand colors if conflicting) |
| `creative_strategy` | Informs composition focus |
| `hook` | Informs what to make visually dominant |
| `adapted_headline` | Text overlay content (if applicable) |

Always **prioritize brand_profile colors and tone** over the competitor's. The structure is borrowed; the identity is the user's.

---

## Regeneration guidelines

If the user rejects a result:
1. First ask: is it composition, colors, style, or text overlay?
2. Adjust only the failing element — do not rewrite the full prompt
3. Log what changed between versions
4. If second attempt also fails: flag to user with both versions and ask for direction
