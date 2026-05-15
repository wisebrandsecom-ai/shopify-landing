---
name: product-research
description: Busca productos físicos de ecommerce con tracción en Meta Ad Library. Evalúa según criterios de producto ganador. Exporta a Excel con scoring. Activar con /product-research.
compatibility: claude-code-only
---

## Preguntas iniciales

Hacer estas preguntas antes de empezar:

1. **¿En qué nicho o categoría quieres buscar?** (o "todos" para búsqueda abierta con keywords generales)
2. **¿Hay nichos que quieras excluir?** (o "ninguno")
3. **¿En qué país buscas?** (recomendado: US — luego UK, DE, FR, IT, ES)
4. **¿Mínimo de ads activos?** (recomendado: 40)
5. **¿Mínimo de días corriendo los ads?** (recomendado: 7)
6. **¿Cuántos productos quieres en el Excel?** (ej. 20, 50)
7. **¿API o Chrome?**
   - *API* → rápido, requiere token en `~/Ad Library/.env`
   - *Chrome* → visual, sin token

```
PAIS          = US | UK | ES | MX | ...
NICHO         = keyword(s) o "todos"
EXCLUIR       = nichos excluidos (o vacío)
MIN_ADS       = número mínimo de ads activos (default 40)
MIN_DAYS      = días mínimos corriendo (default 7)
MAX_RESULTS   = cuántos productos en el Excel
METHOD        = api | chrome
```

---

## ⛔ REGLAS HARDCODED

1. **SOLO ECOMMERCE FÍSICO** — Excluir: servicios, SaaS, apps, agencias, infoproductos, cursos, coaching, digitales. Si hay duda → excluir.

2. **SOLO SHOPIFY** — Solo incluir tiendas Shopify. Detectar visitando el checkout: si muestra el checkout nativo de Shopify (con logo "Shop" o URL `/checkouts/`) → válido. Funnelish, GemPages, PageFly son válidos solo si corren sobre Shopify. Tiendas con checkout completamente custom → excluir.

3. **EXCLUIR MARCAS GRANDES** — Si la tienda tiene Shopify Plus (checkout customizado con elementos extras, URL de marca propia en checkout) → probablemente marca establecida, no dropshipping → excluir.

4. **PDP LINK = destino real del ad** — Seguir el CTA del ad. Idealmente debe llevar a una product page, no a una home. Si lleva a una home → anotar pero marcar como "Home landing".

5. **NO INVENTAR DATOS** — Si no puedes verificar un campo, dejar en blanco.

---

## Keywords de búsqueda

Usar estas keywords para buscar en Meta Ad Library. Combinarlas entre sí y con el NICHO del usuario. Probar en el idioma del país objetivo (traducir con IA, no traductor literal).

**Urgency / Offer:**
```
60/90/30 day guarantee · get yours here · order now
sale ends soon/today/tomorrow/at midnight · limited stock
buy 1 get 1/2 free · risk free · limited stock/limited time
today only · last chance
```

**Social proof / Trust:**
```
happy customers · clinically proven · doctor recommended
why doctors · life changing · game changer · instant results
```

**Pain / Problem:**
```
chronic pain · embarrassing · struggling with · stop suffering
tired of · stop hiding · no more hiding · say goodbye to
the truth about · bed · ed · confidence
```

**Outcome / Desire:**
```
regain your · unstoppable · youthful glow · discreet
for men · for women · natural ingredients
```

**Animal:**
```
pet · dog · cat
```

**Keyword framework** (combinar categorías para el nicho específico):
- Product keyword + Problem keyword (ej. "crema anti arrugas" + "arrugas")
- Mechanism keyword + Outcome keyword (ej. "ácido hialurónico" + "piel joven")
- Pain keyword sola (ej. "chronic pain", "embarrassing")

Generar 5-8 combinaciones de keywords por búsqueda. Si el nicho es específico, añadir keywords del nicho en el idioma local.

---

## Criterios de producto ganador

Estos criterios se aplican para evaluar y puntuar cada producto encontrado.

### Criterios eliminatorios (si falla alguno → descartar automáticamente)

- **Competidores activos escalando** → verificado por MIN_ADS activos
- **Producto físico y evergreen** → no estacional, no tendencia puntual
- **Solo ecommerce** → no servicios ni digitales
- **Tienda Shopify** → verificado en checkout

### Criterios de scoring (0-10 por producto)

Evaluar cada criterio y asignar puntos. El total define el Winner Score.

**1. Soluciona problema o inseguridad grave** ← MÁS IMPORTANTE (0-4 puntos)
- 4pts: problema grave, recurrente, emocional (dolor, salud, inseguridad visible)
- 2pts: problema moderado, mejora algo pero no urgente
- 0pts: producto "bonito" o de tendencia sin problema real detrás
- Señales positivas: copy del ad usa palabras de dolor ("tired of", "stop hiding", "chronic", "embarrassing")
- Señales negativas: copy puramente aspiracional sin problema concreto, producto decorativo

**2. TAM grande** (0-2 puntos)
- 2pts: mercado masivo (salud general, belleza, mascotas, fitness, hombres, mujeres)
- 1pt: mercado medio (un deporte concreto pero popular, un problema común)
- 0pts: nicho muy específico (golf en un solo país, coleccionismo, hobby muy pequeño)

**3. Valor percibido alto** (0-2 puntos)
- 2pts: el producto parece valer más de lo que cuesta, aspecto premium, electrónico o tecnológico
- 1pt: valor percibido medio, producto funcional pero no impresionante visualmente
- 0pts: producto simple, parece barato a primera vista (trocito de plástico, parche)

**4. Fácil de entender** (0-2 puntos)
- 2pts: con solo ver el creativo o la PDP está claro qué hace, cómo funciona y para quién es
- 1pt: necesita algo de explicación pero no mucho
- 0pts: requiere proceso de educación del cliente, concepto complejo

### Winner Score total (0-10)
```
8-10 → 🟢 Producto ganador — testear prioritariamente
5-7  → 🟡 Interesante — hacer más research antes de decidir
0-4  → 🔴 No cumple criterios — descartar o guardar solo como inspiración
```

---

## Camino A — API

### 1. Verificar credenciales

Leer `~/Ad Library/.env`. Si no existe:
> "Crea `~/Ad Library/.env` con:
> ```
> META_ACCESS_TOKEN=tu_token_aqui
> ```"

### 2. Generar keywords

A partir de NICHO + lista de keywords, generar 5-8 combinaciones relevantes.

### 3. Buscar en Meta Ad Library API

Para cada keyword:
```python
params = {
    "access_token": TOKEN,
    "ad_reached_countries": PAIS,
    "ad_active_status": "ACTIVE",
    "search_terms": keyword,
    "ad_type": "ALL",
    "fields": "id,page_id,page_name,ad_delivery_start_time,ad_creative_link_captions,ad_creative_link_titles,ad_snapshot_url",
    "limit": 500,
}
```

Paginar con `after`. Deduplicar por `page_id`.

### 4. Filtrar por MIN_ADS y MIN_DAYS

```python
params_count = {
    "access_token": TOKEN,
    "ad_reached_countries": PAIS,
    "ad_active_status": "ACTIVE",
    "search_page_ids": page_id,
    "fields": "id,ad_delivery_start_time",
    "limit": 500,
}
# Descartar si < MIN_ADS
# Días corriendo: today - min(ad_delivery_start_time)
# Descartar si < MIN_DAYS
```

### 5. Extraer PDP link

Del campo `ad_creative_link_captions` o siguiendo el ad snapshot URL.

---

## Camino B — Chrome MCP

### 1. Navegar a Meta Ad Library

```
https://www.facebook.com/ads/library/?active_status=active&ad_type=all&country={PAIS}&q={KEYWORD}
```

### 2. Aplicar filtros

- Anuncios activos: siempre activado
- Fecha de inicio: desde hace MIN_DAYS días (para garantizar que llevan corriendo)
- Ir variando entre vídeo, imagen, todos — para ver más resultados

### 3. Extraer resultados

- Scroll con `window.scrollBy(0, 1000)` para cargar más
- Para cada ad: nombre de página, fecha inicio, link del ad
- Si el ad tiene "N anuncios usan este contenido" → señal positiva (está siendo escalado, duplicado)
- Seguir el CTA del ad para obtener la PDP link

### 4. Filtrar

- Contar ads activos → descartar si < MIN_ADS
- Calcular días desde inicio → descartar si < MIN_DAYS

---

## Verificación Shopify (ÚLTIMO PASO)

Solo para productos que pasaron todos los filtros anteriores.

Visitar PDP link con Chrome y ejecutar:

```javascript
const isShopify = !!(
  window.Shopify ||
  document.querySelector('link[href*="cdn.shopify.com"]') ||
  document.querySelector('script[src*="cdn.shopify.com"]') ||
  document.querySelector('meta[name="shopify-checkout-api-token"]')
);

const isShopifyPlus = !!(
  document.querySelector('.shopify-plus') ||
  (window.Shopify && window.Shopify.shop && document.querySelector('[data-shopify-plus]'))
);

return JSON.stringify({ isShopify, isShopifyPlus });
```

Si no detectado via JS → añadir al carrito y ver el checkout:
- URL `/checkouts/` + logo "Shop" o elementos nativos Shopify → ✅ válido
- Checkout completamente custom sin elementos Shopify → ❌ excluir
- Shopify Plus (checkout con muchas customizaciones extra, marca propia) → ❌ excluir (marca grande)

---

## Competition Score (1-10)

Contar marcas distintas encontradas anunciando ese mismo producto o similares con las mismas keywords:

```
1-2  → < 5 marcas    🟢 Muy poco explorado
3-4  → 5-15 marcas   🟢 Oportunidad con tracción
5-6  → 16-40 marcas  🟡 Mercado establecido
7-8  → 41-100 marcas 🔴 Muy competido
9-10 → > 100 marcas  🔴 Saturado
```

---

## Exportar a Excel

Guardar en `~/Downloads/product-research_{NICHO}_{PAIS}_{fecha}.xlsx`

Columnas:
```
A:  #
B:  Competitor / Product Name
C:  Product Description
D:  PDP Link               ← hyperlink "Ver PDP"
E:  Ads Library Link       ← hyperlink "Ver Ads"
F:  Platform               ← Shopify / Shopify Plus
G:  Active Ads
H:  Days Running
I:  Date Found
J:  Keywords Used
K:  Competition Score      ← 1-10 + emoji
L:  Winner Score           ← 0-10 + emoji (🟢/🟡/🔴)
M:  Score Breakdown        ← Problema(x/4) TAM(x/2) Valor(x/2) Fácil(x/2)
```

```python
import openpyxl
from openpyxl.styles import Font, PatternFill, Alignment, Border, Side
from datetime import date
import os

wb = openpyxl.Workbook()
ws = wb.active
ws.title = "Product Research"

headers = ["#", "Competitor / Product Name", "Product Description", "PDP Link",
           "Ads Library Link", "Platform", "Active Ads", "Days Running",
           "Date Found", "Keywords Used", "Competition Score",
           "Winner Score", "Score Breakdown"]

header_fill = PatternFill("solid", fgColor="1877F2")
header_font = Font(bold=True, color="FFFFFF", size=11)

for col, h in enumerate(headers, 1):
    cell = ws.cell(row=1, column=col, value=h)
    cell.fill = header_fill
    cell.font = header_font
    cell.alignment = Alignment(horizontal="center", vertical="center", wrap_text=True)

fill_par   = PatternFill("solid", fgColor="EBF2FF")
fill_impar = PatternFill("solid", fgColor="FFFFFF")

for i, p in enumerate(products, 1):
    row = i + 1
    fill = fill_par if i % 2 == 0 else fill_impar
    ws.cell(row=row, column=1,  value=i).fill = fill
    ws.cell(row=row, column=2,  value=p["name"]).fill = fill
    ws.cell(row=row, column=3,  value=p["description"]).fill = fill

    pdp = ws.cell(row=row, column=4, value="Ver PDP")
    pdp.hyperlink = p["pdp_link"]
    pdp.font = Font(color="1877F2", underline="single")
    pdp.fill = fill

    lib = ws.cell(row=row, column=5, value="Ver Ads")
    lib.hyperlink = p["ads_library_link"]
    lib.font = Font(color="1877F2", underline="single")
    lib.fill = fill

    ws.cell(row=row, column=6,  value=p["platform"]).fill = fill
    ws.cell(row=row, column=7,  value=p["active_ads"]).fill = fill
    ws.cell(row=row, column=8,  value=p["days_running"]).fill = fill
    ws.cell(row=row, column=9,  value=str(date.today())).fill = fill
    ws.cell(row=row, column=10, value=p["keywords"]).fill = fill
    ws.cell(row=row, column=11, value=p["competition_score"]).fill = fill
    ws.cell(row=row, column=12, value=p["winner_score"]).fill = fill
    ws.cell(row=row, column=13, value=p["score_breakdown"]).fill = fill

thin = Side(style="thin", color="CCCCCC")
border = Border(left=thin, right=thin, top=thin, bottom=thin)
for row in ws.iter_rows(min_row=1, max_row=ws.max_row, max_col=13):
    for cell in row:
        cell.border = border
        if not cell.hyperlink:
            cell.alignment = Alignment(horizontal="left", vertical="center", wrap_text=True)

col_widths = [5, 28, 35, 10, 10, 14, 12, 14, 14, 22, 18, 14, 30]
cols = ["A","B","C","D","E","F","G","H","I","J","K","L","M"]
for col, width in zip(cols, col_widths):
    ws.column_dimensions[col].width = width

ws.row_dimensions[1].height = 30
for r in range(2, ws.max_row + 1):
    ws.row_dimensions[r].height = 20

ws.freeze_panes = "A2"

output = os.path.expanduser(f"~/Downloads/product-research_{NICHO}_{PAIS}_{date.today()}.xlsx")
wb.save(output)
```

Abrir automáticamente:
```bash
open ~/Downloads/product-research_{NICHO}_{PAIS}_{fecha}.xlsx
```

---

## Confirmación final

```
========================================
  PRODUCT RESEARCH COMPLETE
========================================
País:              {PAIS}
Nicho:             {NICHO}
Keywords usadas:   {lista}

Encontrados:       {N} productos
Descartados:
  - Por MIN_ADS:   {N}
  - Por MIN_DAYS:  {N}
  - No Shopify:    {N}
  - No ecommerce:  {N}

En Excel:          {N} productos
  🟢 Winner:       {N}
  🟡 Interesante:  {N}
  🔴 Descartar:    {N}

Archivo: ~/Downloads/product-research_{NICHO}_{PAIS}_{fecha}.xlsx
========================================
```
