---
name: product-research
description: Busca productos físicos de ecommerce con tracción en Meta Ad Library. Filtra por ads activos, días corriendo, plataforma (Shopify/Funnelish/GemPages/PageFly) y nivel de competencia. Exporta a Excel. Activar con /product-research.
compatibility: claude-code-only
---

## Preguntas iniciales

Hacer estas preguntas antes de empezar:

1. **¿En qué nicho o categoría quieres buscar?** (ej. suplementos masculinos, pet care, skincare) — o escribe "todos" para búsqueda abierta
2. **¿Hay nichos que quieras excluir?** (ej. ropa, electrónica, servicios digitales) — o "ninguno"
3. **¿En qué país buscas?** (recomendado: US)
4. **¿Mínimo de ads activos?** (recomendado: 40)
5. **¿Mínimo de días corriendo los ads?** (recomendado: 7)
6. **¿Precio mínimo o máximo del producto?** — o "sin filtro"
7. **¿Cuántos productos quieres en el Excel?** (ej. 20, 50)
8. **¿API o Chrome?**
   - *API* → rápido, masivo, requiere token en `~/Ad Library/.env`
   - *Chrome* → visual, sin token

Guardar respuestas:
```
PAIS          = US | ES | MX | ...
NICHO         = keyword(s) de búsqueda
EXCLUIR       = nichos excluidos (o vacío)
MIN_ADS       = número mínimo de ads activos (default 40)
MIN_DAYS      = días mínimos corriendo (default 7)
PRECIO_MIN    = número o null
PRECIO_MAX    = número o null
MAX_RESULTS   = cuántos productos en el Excel
METHOD        = api | chrome
```

---

## ⛔ REGLAS HARDCODED

1. **SOLO ECOMMERCE FÍSICO** — Excluir automáticamente: servicios, SaaS, apps, agencias, infoproductos, cursos, coaching, productos digitales. Si hay duda → excluir.

2. **SOLO PLATAFORMAS PERMITIDAS** — Solo incluir productos cuya landing usa: Shopify, Funnelish, GemPages o PageFly. Verificar siempre DESPUÉS de pasar todos los demás filtros (último paso).

3. **PDP LINK = destino real del ad** — No el perfil del anunciante. Seguir el CTA del ad y capturar la URL de aterrizaje.

4. **NO INVENTAR DATOS** — Si no puedes verificar un campo, dejar en blanco. Nunca estimar sin base.

---

## Camino A — API

### 1. Verificar credenciales

Leer `~/Ad Library/.env`. Si no existe:
> "Crea `~/Ad Library/.env` con:
> ```
> META_ACCESS_TOKEN=tu_token_aqui
> ```"

### 2. Construir keywords de búsqueda

A partir de NICHO, generar 3-5 keywords relacionadas para maximizar cobertura:
```
NICHO = "suplementos masculinos"
KEYWORDS = ["testosterone supplement", "male enhancement", "men vitality", "ED supplement", "capsaicin men"]
```

### 3. Buscar ads en Meta Ad Library API

Para cada keyword:
```python
params = {
    "access_token": TOKEN,
    "ad_reached_countries": PAIS,
    "ad_active_status": "ACTIVE",
    "search_terms": keyword,
    "ad_type": "ALL",
    "fields": "id,page_id,page_name,ad_delivery_start_time,ad_creative_link_captions,ad_creative_link_descriptions,ad_creative_link_titles,ad_snapshot_url",
    "limit": 500,
}
```

Paginar con cursor `after`. Deduplicar por `page_id`.

### 4. Filtrar por MIN_ADS activos

Para cada página encontrada, contar sus ads activos:
```python
params_count = {
    "access_token": TOKEN,
    "ad_reached_countries": PAIS,
    "ad_active_status": "ACTIVE",
    "search_page_ids": page_id,
    "fields": "id,ad_delivery_start_time",
    "limit": 500,
}
```
- Contar ads activos totales → descartar si < MIN_ADS
- Calcular días corriendo: `(today - min(ad_delivery_start_time)).days` → descartar si < MIN_DAYS

### 5. Extraer PDP link

Del campo `ad_creative_link_captions` o `ad_snapshot_url`, extraer la URL de destino del ad.
Si no disponible vía API → marcar para visita Chrome en paso de verificación.

---

## Camino B — Chrome MCP

### 1. Construir URL de búsqueda

```
https://www.facebook.com/ads/library/?active_status=active&ad_type=all&country={PAIS}&q={KEYWORD}&media_type=all
```

### 2. Navegar y extraer

- `navigate` a la URL
- Aplicar filtro "Ordenar por: Más recientes" para ver ads activos primero
- Scroll con `window.scrollBy(0, 1000)` para cargar más resultados
- Para cada ad card extraer:
  - Nombre de la página
  - Fecha de inicio del ad
  - Link "Ver detalles del anuncio" → extraer URL de destino del CTA

### 3. Filtrar

- Contar ads activos por página → descartar si < MIN_ADS
- Calcular días desde inicio → descartar si < MIN_DAYS

---

## Filtro de plataforma (ÚLTIMO PASO — aplicar solo a los que pasaron todos los filtros)

Para cada producto que pasa los filtros anteriores, visitar la PDP link con Claude in Chrome:

```
navigate(url: PDP_LINK)
```

Inspeccionar con `javascript_tool`:
```javascript
// Shopify
const isShopify = !!(
  window.Shopify ||
  document.querySelector('link[href*="cdn.shopify.com"]') ||
  document.querySelector('script[src*="cdn.shopify.com"]') ||
  document.querySelector('meta[name="shopify-checkout-api-token"]')
);

// GemPages
const isGemPages = !!(
  document.querySelector('[class*="gempages"]') ||
  document.querySelector('script[src*="gempages"]') ||
  window.__gempages__
);

// PageFly
const isPageFly = !!(
  document.querySelector('[class*="pagefly"]') ||
  document.querySelector('script[src*="pagefly"]')
);

// Funnelish
const isFunnelish = !!(
  window.Funnelish ||
  document.querySelector('script[src*="funnelish"]') ||
  document.querySelector('[data-funnelish]')
);

// Checkout URL check (Shopify fallback)
const checkoutLinks = Array.from(document.querySelectorAll('a[href*="/checkout"]'));
const hasShopifyCheckout = checkoutLinks.some(l => l.href.includes('.myshopify.com') || l.href.includes('/checkout'));

return JSON.stringify({ isShopify, isGemPages, isPageFly, isFunnelish, hasShopifyCheckout });
```

- Si ninguna plataforma detectada → descarta el producto
- Si detecta Shopify, GemPages, PageFly o Funnelish → incluir, anotar plataforma

**También verificar que es ecommerce físico:**
- ¿Hay botón de "Add to Cart" / "Buy Now"?
- ¿Hay precio visible?
- ¿Es un producto tangible (no servicio, no digital)?
- Si hay duda → excluir

---

## Nivel de competencia (escala 1-10)

Para cada producto incluido, calcular score de competencia:

```
MARCAS_ACTIVAS = número de páginas distintas encontradas anunciando ese producto o similar
                 (usando las mismas keywords de búsqueda)

SCORE:
  1-2  → < 5 marcas activas       (nicho muy poco explorado)
  3-4  → 5-15 marcas activas      (oportunidad con algo de tracción)
  5-6  → 16-40 marcas activas     (mercado establecido, competencia media)
  7-8  → 41-100 marcas activas    (muy competido)
  9-10 → > 100 marcas activas     (saturado)
```

Añadir columna `Competition Score` con número + etiqueta:
- 1-3: 🟢 Baja
- 4-6: 🟡 Media
- 7-10: 🔴 Alta

---

## Exportar a Excel

Guardar en `~/Downloads/product-research_{NICHO}_{PAIS}_{fecha}.xlsx`

Columnas:
```
A: #
B: Competitor / Product Name    ← nombre de la marca anunciante
C: Product Description          ← qué es el producto (extraído del ad copy o PDP)
D: PDP Link                     ← URL de aterrizaje del ad
E: Ads Library Link             ← https://www.facebook.com/ads/library/?view_all_page_id={page_id}
F: Platform                     ← Shopify / GemPages / PageFly / Funnelish
G: Active Ads                   ← número de ads activos
H: Days Running                 ← días desde el primer ad activo
I: Date Found                   ← fecha de hoy
J: Keywords Used                ← keyword(s) que encontraron este producto
K: Competition Score            ← número 1-10 + emoji
```

```python
import openpyxl
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from datetime import date

wb = openpyxl.Workbook()
ws = wb.active
ws.title = "Product Research"

headers = ["#", "Competitor / Product Name", "Product Description", "PDP Link",
           "Ads Library Link", "Platform", "Active Ads", "Days Running",
           "Date Found", "Keywords Used", "Competition Score"]

header_fill = PatternFill("solid", fgColor="1877F2")
header_font = Font(bold=True, color="FFFFFF", size=11)

for col, h in enumerate(headers, 1):
    cell = ws.cell(row=1, column=col, value=h)
    cell.fill = header_fill
    cell.font = header_font
    cell.alignment = Alignment(horizontal="center", vertical="center", wrap_text=True)

fill_par   = PatternFill("solid", fgColor="EBF2FF")
fill_impar = PatternFill("solid", fgColor="FFFFFF")

for i, product in enumerate(products, 1):
    row = i + 1
    fill = fill_par if i % 2 == 0 else fill_impar

    ws.cell(row=row, column=1,  value=i).fill = fill
    ws.cell(row=row, column=2,  value=product["name"]).fill = fill
    ws.cell(row=row, column=3,  value=product["description"]).fill = fill

    pdp = ws.cell(row=row, column=4, value="Ver PDP")
    pdp.hyperlink = product["pdp_link"]
    pdp.font = Font(color="1877F2", underline="single")
    pdp.fill = fill

    lib = ws.cell(row=row, column=5, value="Ver Ads")
    lib.hyperlink = product["ads_library_link"]
    lib.font = Font(color="1877F2", underline="single")
    lib.fill = fill

    ws.cell(row=row, column=6,  value=product["platform"]).fill = fill
    ws.cell(row=row, column=7,  value=product["active_ads"]).fill = fill
    ws.cell(row=row, column=8,  value=product["days_running"]).fill = fill
    ws.cell(row=row, column=9,  value=str(date.today())).fill = fill
    ws.cell(row=row, column=10, value=product["keywords"]).fill = fill
    ws.cell(row=row, column=11, value=product["competition_score"]).fill = fill

thin = Side(style="thin", color="CCCCCC")
border = Border(left=thin, right=thin, top=thin, bottom=thin)
for row in ws.iter_rows(min_row=1, max_row=ws.max_row, max_col=11):
    for cell in row:
        cell.border = border
        if not cell.hyperlink:
            cell.alignment = Alignment(horizontal="left", vertical="center", wrap_text=True)

col_widths = [5, 30, 40, 12, 12, 14, 12, 14, 14, 25, 18]
cols = ["A","B","C","D","E","F","G","H","I","J","K"]
for col, width in zip(cols, col_widths):
    ws.column_dimensions[col].width = width

ws.row_dimensions[1].height = 30
for r in range(2, ws.max_row + 1):
    ws.row_dimensions[r].height = 20

ws.freeze_panes = "A2"

output = f"~/Downloads/product-research_{NICHO}_{PAIS}_{date.today()}.xlsx"
wb.save(os.path.expanduser(output))
```

Abrir automáticamente:
```bash
open ~/Downloads/product-research_{NICHO}_{PAIS}_{fecha}.xlsx
```

---

## Confirmación final

Imprimir:
- N productos encontrados
- N descartados por plataforma
- N descartados por MIN_ADS
- N descartados por MIN_DAYS
- Ruta del Excel
