# Copy File Format — Native Ads

One file per ad: `copy_nativa_0N.txt`

⚠️ Never combine all copies into one file. N ads = N separate files.

---

## Format

Match the structure of the input copy exactly.
Same field labels. Same paragraph count. Same line breaks.

Example:
```
Primary text: Llevaba años sintiéndome agotado y sin energía.

Mi médico me dijo que era "normal para mi edad" pero yo sabía que algo estaba mal.

Entonces encontré esto y en tres semanas noté la diferencia.

Headline: Lo que nadie te dice sobre la energía después de los 40

Description: Envío gratis a todo México
```

---

## Rules

- OUTPUT_LANGUAGE only — zero words from source language
- Localized to OUTPUT_LOCALE — natural vocabulary for that country, not neutral textbook Spanish
- Same paragraph breaks as original — if original has 6 paragraphs, output has 6
- Same sentence rhythm — short punchy sentences stay short. Long flowing sentences stay long
- Same marketing structure — hook position, problem framing, CTA placement unchanged
- No em dashes (—), no en dashes (–), no hyphens used as separators
- No bullet points unless original has them
- No added transitions ("Furthermore", "Additionally", "In conclusion")
- No AI preamble ("Here is the adapted copy...")
- No improvements — translate/recreate, do not edit
- Storytelling copy is long by design — do not shorten it
- **WORD COUNT CHECK** — Before saving, count source paragraphs and output paragraphs. They must match exactly. If they don't, finish the missing paragraphs before saving.
- Never end copy early with "..." or "[continues]" or any truncation marker
