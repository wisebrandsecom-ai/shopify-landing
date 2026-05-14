# Drive Upload — Reference

---

## Folder structure

Flat — no subfolders per ad:

```
Desktop:
  OUTPUT_FOLDER/                ← e.g. "Batch 001 Output" on desktop
    anuncio_01.png              ← downloaded from Higgsfield CDN
    copy_anuncio_01.txt         ← adapted copy in OUTPUT_LANGUAGE
    anuncio_02.png
    copy_anuncio_02.txt

Drive:
  DRIVE_PARENT/                 ← you created this (e.g. "Creativos EPB")
    DRIVE_BATCH/                ← Claude creates this (e.g. "batch 001")
      anuncio_01.png
      copy_anuncio_01.txt
      anuncio_02.png
      copy_anuncio_02.txt
```

---

## Step 1 — Download images locally

```bash
curl -L "HIGGSFIELD_CDN_URL" -o "/path/to/ADS_FOLDER - Output/AD N/ad_0N_static.png"
```

- Full resolution, no compression, no resizing
- One curl call per ad
- Verify file exists and is > 100KB after download

---

## Step 2 — Write copy.txt

One file per ad. Format matches the input copy.txt the user provides.
Only OUTPUT_LANGUAGE text. No metadata, no headers, no dashes (em dashes or hyphens between phrases).

Format:
```
Primary text: [adapted primary text]

Headline: [adapted headline]

Description: [adapted description]
```

Rules:
- Same labels as input copy.txt (Primary text, Headline, Description, CTA — whatever the input uses)
- Same paragraph structure as input
- No added dashes or em dashes
- No AI filler phrases
- Same tone and register as the source

---

## Step 3 — Upload to Drive via Claude in Chrome

Use Claude_in_Chrome MCP to upload files from OUTPUT_FOLDER to Drive:

1. Navigate to Google Drive: `navigate(url: "https://drive.google.com")`
2. Find DRIVE_PARENT folder (search by name if needed)
3. Create DRIVE_BATCH folder inside it
4. Open DRIVE_BATCH folder
5. For each file in OUTPUT_FOLDER, use `file_upload` tool:
   ```
   file_upload(path: "OUTPUT_FOLDER/anuncio_01.png")
   file_upload(path: "OUTPUT_FOLDER/copy_anuncio_01.txt")
   file_upload(path: "OUTPUT_FOLDER/anuncio_02.png")
   file_upload(path: "OUTPUT_FOLDER/copy_anuncio_02.txt")
   ```
6. Take a screenshot to confirm all files are uploaded
7. Return the Drive batch folder URL

No base64. No MCP file limits. Chrome uploads directly from disk.

---

## File naming

Images: `ad_[n]_[static|native].png`
Copy: `copy.txt` (always this name, inside each ad subfolder)
