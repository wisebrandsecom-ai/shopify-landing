# Soul 1.0 Prompting — Native Ads

Model: `soul_1` | Style: `realistic` | Count: always 3

---

## Core principle

The goal is a photo that looks like a real person took it on their phone.
Not a professional photographer. Not a studio shot. Not an influencer.
A regular person, in a real place, living their life.

---

## What makes a native image

✅ Looks native:
- Slightly imperfect framing (not perfectly centered)
- Natural indoor or outdoor light (window, overcast, warm kitchen lamp)
- Real backgrounds with some clutter or depth
- Person in natural pose (sitting, standing casually, looking at camera or away)
- Clothing is ordinary (t-shirt, casual top, everyday outfit)
- Expression is real — relaxed, genuine smile, neutral, slightly tired
- Shot from eye level or slightly below — how a person holds their phone

❌ Looks generated / reject:
- Perfect symmetrical lighting
- Too-smooth skin or hair
- Extra fingers, merged hands, floating limbs
- Exaggerated expression (over-smiling, dramatic face)
- Perfect composition (rule of thirds, professional framing)
- Backgrounds that look like stock photos
- Clothing or setting that looks "curated" or styled
- Any element that makes you think "AI made this"

---

## Prompt structure

```
[person description: age range, gender, rough appearance, clothing],
[natural action: what they're doing],
[setting: specific real location with details],
[lighting: natural light source],
[camera feel: how it was taken],
candid, unpolished, authentic, phone photo, not professional photography
```

---

## Examples

**Good prompt (lifestyle/health):**
```
Woman in her late 40s, brown hair pulled back, wearing a grey cotton t-shirt,
sitting at a round wooden kitchen table with a coffee mug in front of her,
holding a small supplement bottle, looking directly at camera with a relaxed genuine smile,
morning light from a window to her left, slightly cluttered kitchen counter in background,
eye-level phone selfie, candid, natural, not posed
```

**Good prompt (before/after feel):**
```
Man in his mid-50s, slightly overweight, grey stubble, wearing a navy blue polo,
standing in a living room near a couch, arms at sides, looking at camera with a calm expression,
warm lamp light from the right, bookshelves visible in background slightly out of focus,
phone photo taken by someone standing in front of him, real home environment, candid
```

**Good prompt (outdoor casual):**
```
Woman in her early 40s, light skin, wavy dark hair, wearing a white linen blouse,
sitting on a park bench with trees behind her, looking slightly off-camera, relaxed expression,
overcast natural daylight, soft even light with no harsh shadows,
slightly angled phone shot, spontaneous, not posed, authentic
```

---

## What to avoid in prompts

- "beautiful", "stunning", "gorgeous", "perfect" — attracts AI aesthetics
- "professional photography", "studio", "editorial" — wrong direction
- "dramatic lighting", "golden hour" — too polished
- Highly specific poses ("hand on chin", "crossed arms") — often renders badly
- "full body shot" — increases limb distortion risk; stick to waist-up or face/torso

---

## After generation: selection criteria

View all 3 variations. Ask yourself:
"Could this person have posted this on their personal Facebook?"

If yes → candidate.
If no → reject.

Pick the most believable one. If none pass → regenerate with a simpler, more grounded prompt.
