# Connection — Reference

Loaded during Step 0 of the shopify-landing skill.

---

## Standard connection prompt (give this to clients)

```
STORE_URL: mi-tienda.myshopify.com
OAUTH_CLIENT: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
OAUTH_SECRET: shpss_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

1. Ejecuta este comando para autenticarte:
shopify store auth --store STORE_URL --scopes write_products,write_inventory,write_themes,write_content,write_files,write_metaobjects,write_metaobject_definitions,write_publications,write_translations,read_locations,read_analytics

   Se abrirá el navegador — aprueba la autorización y vuelve.

2. Confírmame cuando hayas aprobado en el navegador para continuar.
```

---

## What Claude does when it receives this prompt

1. Extract `STORE_URL`, `OAUTH_CLIENT`, and `OAUTH_SECRET` — store as session variables.
2. Run:
```bash
shopify store auth --store STORE_URL --scopes write_products,write_inventory,write_themes,write_content,write_files,write_metaobjects,write_metaobject_definitions,write_publications,write_translations,read_locations,read_analytics
```
3. Tell the user: "Se abrirá el navegador — aprueba la autorización y vuelve."
4. Wait for user to confirm, then verify the connection:
```bash
shopify store execute -s STORE_URL -q '{ shop { name plan { displayName } currencyCode } }'
```
5. Confirm:
```
✅ Conectado a: [shop.name]
   Plan: [plan.displayName] | Moneda: [currencyCode]
```

---

## How authentication works

After `shopify store auth` is approved in the browser, the CLI stores an **online access token** locally. All subsequent operations use this stored token.

**For GraphQL queries and mutations** — use the CLI:
```bash
shopify store execute -s STORE_URL -q 'YOUR_GRAPHQL_QUERY'
```

**For REST API calls** (assets, themes, files) — use the access token from the CLI session. The correct header is:
```bash
-H "X-Shopify-Access-Token: <token_from_cli_session>"
```

> ⚠️ `X-Shopify-Client-Id` and `X-Shopify-Client-Secret` are NOT valid REST authentication headers. They are OAuth app credentials used only during the OAuth flow, not for API calls. Never use them as request headers.

`OAUTH_CLIENT` and `OAUTH_SECRET` are stored in context for reference but are not used directly in API calls after auth is complete.

---

## Required app scopes

Shopify Admin → Settings → Apps and sales channels → Develop apps → Create an app → Configure Admin API scopes:

```
write_products
write_inventory
write_themes
write_content
write_files
write_metaobjects
write_metaobject_definitions
write_publications
write_translations
read_locations
read_analytics
```

---

## CLI version note

Tested against Shopify CLI 3.x:
- ✅ `shopify store auth --store <url> --scopes <scopes>`
- ✅ `shopify store execute -s <url> -q '<graphql>'`
- ❌ `shopify auth login --store` (does not exist in 3.x)
- ❌ `shopify store info` (does not exist in 3.x)

---

## Full landing prompt template (paste this to start)

```
STORE_URL: mi-tienda.myshopify.com
OAUTH_CLIENT: xxxxxxxxxxxxxxxx
OAUTH_SECRET: shpss_xxxxxxxxxxxxxxxx

COMPETITOR_URL: https://competidor.com/products/producto  ← URL real de la página de producto
OUTPUT_LANGUAGE: es
BRAND_NAME: Tu Marca  ← tu nombre de marca (reemplaza el del competidor en todos los textos)
TEMPLATE_NAME: nombre-producto
THEME_MODE: A
COLOR_PRIMARY: #1a4731
COLOR_TEXT: #111111
COPY_STRATEGY: B
VISUAL_MODE: REPLICATE
PRODUCT_PHOTOS_FOLDER: /Users/tu-usuario/Desktop/fotos-producto  ← carpeta local con fotos de referencia para el producto

[adjunta el PDF del competidor]
[adjunta la foto del producto PNG con fondo transparente]

Ejecuta el skill completo de principio a fin sin pausas ni confirmaciones.
Incluye Higgsfield automáticamente al final usando la foto del producto adjunta.
```

**COMPETITOR_URL:** Claude abre la web con la extensión de Chrome para ver la página visualmente — animaciones, fuentes, spacing, mobile view. Dar la URL del producto específico.

**PRODUCT_PHOTOS_FOLDER:** Carpeta en tu escritorio con imágenes de referencia de composición (screenshots del competidor u otras que quieras replicar). Claude las lee, analiza la composición de cada una, y genera fotos de tu producto siguiendo esa composición en Higgsfield. No se necesita Drive para nada.

**Notes:**
- THEME_MODE A = duplicate base theme (recommended)
- COPY_STRATEGY B = translate competitor copy to OUTPUT_LANGUAGE
- Attach product PNG → Claude uploads to Higgsfield and uses it for ALL product shots
- "sin confirmaciones" = no intermediate approval needed, run to completion
