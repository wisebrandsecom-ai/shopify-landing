# Copy File Format

One file per ad: `copy_anuncio_0N.txt`

⚠️ Never write a single combined file with all copies. N ads = N separate files.

## Format

Match the structure of the input copy.txt exactly.
Use the same field labels. No extra headers, no metadata, no dashes between sections.

Example:
```
Primary text: La disfunción eréctil significa una cosa: arterias obstruidas y mala circulación.

Headline: 50% DE DESCUENTO HOY

Description: Envío gratis
```

## Rules

- OUTPUT_LANGUAGE only — zero source-language words
- Localized to OUTPUT_LOCALE — vocabulary, expressions, and references native to that country
- Same labels as input (Primary text, Headline, Description, CTA...)
- Same paragraph breaks as input — count paragraphs in source, output must match exactly
- **FULL LENGTH MANDATORY** — Never truncate, summarize, or cut for length. If source Primary text is 10 sentences, output is 10 sentences. Verify paragraph count before saving.
- Never end with "..." or "[continues]" or any truncation marker
- No em dashes or hyphens between phrases
- No added commentary or explanations
- No AI preamble ("Here is the translation of...")
- No exaggerated regional slang — natural native register, not a caricature
