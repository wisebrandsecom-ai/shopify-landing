---
name: scraper competidores
description: Busca competidores activos en Meta Ad Library, filtra por mínimo de ads activos configurable, y exporta a Excel con los 5 creativos con más impresiones de cada competidor. Activar con /competidores.
compatibility: claude-code-only
---

## Preguntas iniciales

Hacer estas preguntas antes de empezar:

1. **¿Qué producto vendes y qué problema resuelve?**
2. **¿En qué país quieres buscar?** (ej. España → ES, México → MX, USA → US)
3. **¿Tienes algún competidor ya en mente**, o empezamos desde cero con palabras clave?
4. **¿Cuántos competidores quieres en el resultado final?** (ej. 10, 20, 50)
5. **¿Mínimo de ads activos para incluir un competidor?** (default: 10)
6. **¿Solo imágenes, solo videos, o ambos?**
7. **¿API o Chrome?**
   - *API* → rápido, masivo, requiere token en `~/Ad Library/.env`
   - *Chrome* → navega visualmente, útil sin token

Guardar respuestas:
```
PAIS        = ES | MX | US | ...
NICHO       = keyword del producto
MIN_ADS     = número mínimo de ads activos (default 10)
MAX_RESULTS = cuántos competidores en el Excel
AD_TYPE     = ALL | VIDEO | IMAGE_AND_MEME
METHOD      = api | chrome
```

---

## Camino A — API

### 1. Verificar credenciales

Leer `~/Ad Library/.env`. Si no existe, parar:

> "Crea `~/Ad Library/.env` con:
> ```
> META_ACCESS_TOKEN=tu_token_aqui
> ```"

### 2. Buscar anunciantes

```python
params = {
    "access_token": TOKEN,
    "ad_reached_countries": PAIS,
    "ad_active_status": "ACTIVE",
    "search_terms": NICHO,
    "ad_type": AD_TYPE,
    "fields": "page_id,page_name,ad_delivery_start_time,impressions",
    "limit": 500,
}
```

Paginar con `after` hasta agotar. Agrupar por `page_id`.

### 3. Filtrar por MIN_ADS

Contar ads activos reales de cada página:

```python
params_count = {
    "access_token": TOKEN,
    "ad_reached_countries": PAIS,
    "ad_active_status": "ACTIVE",
    "search_page_ids": page_id,
    "fields": "id",
    "limit": 500,
}
```

Descartar si < MIN_ADS. Parar al alcanzar MAX_RESULTS.

### 4. Top 5 creativos por impresiones

Para cada competidor válido:

```python
params_top = {
    "access_token": TOKEN,
    "ad_reached_countries": PAIS,
    "ad_active_status": "ACTIVE",
    "search_page_ids": page_id,
    "fields": "id,impressions",
    "limit": 50,
}
# Ordenar por impressions.lower_bound desc → tomar top 5
# Link por ad: https://www.facebook.com/ads/library/?id={ad_id}
```

---

## Camino B — Chrome MCP

### 1. Buscar en Ad Library

```
https://www.facebook.com/ads/library/?active_status=active&ad_type={AD_TYPE}&country={PAIS}&q={NICHO}
```

Scroll con `window.scrollBy(0, 1000)` hasta tener suficientes resultados.
Extraer `page_id` y nombre de cada anunciante visible.

### 2. Para cada competidor: abrir perfil + aplicar filtro impresiones

Navegar a:
```
https://www.facebook.com/ads/library/?active_status=active&ad_type=all&country={PAIS}&view_all_page_id={page_id}
```

- Hacer clic en **"Ordenar por"** → seleccionar **"Impresiones: de más a menos"**
- Extraer los primeros 5 ads (mayor impresión): copiar el `id` de "Identificador de la biblioteca"
- Link por ad: `https://www.facebook.com/ads/library/?id={ad_id}`
- Contar total de ads activos en la página

### 3. Filtrar y parar

- < MIN_ADS → descartar
- Parar al alcanzar MAX_RESULTS competidores válidos

---

## Exportar a Excel

Guardar en `~/Downloads/competidores_{NICHO}_{PAIS}.xlsx`

Columnas: `# | Página | Ads activos | Ad Library | Top 1 | Top 2 | Top 3 | Top 4 | Top 5`

- Columna D: hyperlink "Ver todos" → perfil completo del anunciante
- Columnas E–I: hyperlinks "Creativo 1"…"Creativo 5" → links individuales de cada ad

```python
import openpyxl
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side

wb = openpyxl.Workbook()
ws = wb.active
ws.title = "Competidores"

headers = ["#", "Página", "Ads activos", "Ad Library", "Top 1", "Top 2", "Top 3", "Top 4", "Top 5"]
header_fill = PatternFill("solid", fgColor="1877F2")
header_font = Font(bold=True, color="FFFFFF", size=11)
for col, h in enumerate(headers, 1):
    cell = ws.cell(row=1, column=col, value=h)
    cell.fill = header_fill
    cell.font = header_font
    cell.alignment = Alignment(horizontal="center", vertical="center")

fill_par   = PatternFill("solid", fgColor="EBF2FF")
fill_impar = PatternFill("solid", fgColor="FFFFFF")

for i, comp in enumerate(competidores, 1):
    row = i + 1
    fill = fill_par if i % 2 == 0 else fill_impar
    ws.cell(row=row, column=1, value=i).fill = fill
    ws.cell(row=row, column=2, value=comp["nombre"]).fill = fill
    ws.cell(row=row, column=3, value=comp["ads_activos"]).fill = fill

    c = ws.cell(row=row, column=4, value="Ver todos")
    c.hyperlink = comp["link"]
    c.font = Font(color="1877F2", underline="single")
    c.fill = fill

    for j, ad_link in enumerate(comp.get("top_ads", [])[:5], 5):
        c = ws.cell(row=row, column=j, value=f"Creativo {j-4}")
        c.hyperlink = ad_link
        c.font = Font(color="1877F2", underline="single")
        c.fill = fill

thin = Side(style="thin", color="CCCCCC")
border = Border(left=thin, right=thin, top=thin, bottom=thin)
for row in ws.iter_rows(min_row=1, max_row=ws.max_row, max_col=9):
    for cell in row:
        cell.border = border
        cell.alignment = Alignment(horizontal="left", vertical="center")

ws.column_dimensions["A"].width = 5
ws.column_dimensions["B"].width = 35
ws.column_dimensions["C"].width = 14
ws.column_dimensions["D"].width = 14
for col in ["E","F","G","H","I"]:
    ws.column_dimensions[col].width = 14
ws.row_dimensions[1].height = 22
for r in range(2, ws.max_row + 1):
    ws.row_dimensions[r].height = 18

ws.freeze_panes = "A2"
output_path = os.path.expanduser(f"~/Downloads/competidores_{NICHO}_{PAIS}.xlsx")
wb.save(output_path)
```

## Confirmar

- N competidores encontrados
- Ruta del archivo
- `open ~/Downloads/competidores_{NICHO}_{PAIS}.xlsx`
