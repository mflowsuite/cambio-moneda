# cambio-moneda

Convertidor de monedas responsive en tiempo real entre **AED**, **USD**, **EUR**, **ARS** (dólar blue venta), **BRL** e **ILS**. PWA instalable con soporte **offline**. Single-file vanilla HTML/CSS/JS, deployado en GitHub Pages.

🌐 **Live:** [mflowsuite.github.io/cambio-moneda](https://mflowsuite.github.io/cambio-moneda/)

---

## Características

- **Conversión bidireccional en tiempo real** — al editar cualquier campo, los otros 5 se actualizan instantáneamente
- **Dólar blue venta** desde [Bluelytics](https://bluelytics.com.ar/) (sin API key)
- **AED, EUR, BRL e ILS** desde [open.er-api.com](https://www.exchangerate-api.com/docs/free) (sin API key)
- **Botón "Actualizar tasas"** con spinner, timestamp y manejo de errores
- **PWA instalable** — `manifest.json` para "Agregar a pantalla de inicio" en mobile/desktop
- **Modo offline** — service worker cachea el app shell + `localStorage` guarda las últimas tasas
- **Responsive** — grilla 2×2 en tablet/desktop, 1 columna en mobile
- **Mobile-first** — `inputmode="decimal"` para teclado numérico nativo
- **Accesible** — `prefers-reduced-motion`, focus visible, sin scroll horizontal
- **Zero dependencias** — sin frameworks, sin build process

---

## Stack

| Capa | Tecnología |
|------|------------|
| Frontend | HTML5 + CSS3 + Vanilla JS (ES2017+) |
| Offline | Service Worker + Cache API + localStorage |
| PWA | Web App Manifest |
| Tipografía | Fira Code · Fira Sans (Google Fonts) |
| Banderas | [flagcdn.com](https://flagcdn.com/) |
| Hosting | GitHub Pages |
| Design system | [MflowSuite](https://mflowsuite.com/) |

---

## APIs consumidas

| Dato | API | Endpoint | Campo |
|------|-----|----------|-------|
| Dólar blue venta | Bluelytics | `https://api.bluelytics.com.ar/v2/latest` | `blue.value_sell` |
| AED / EUR / BRL / ILS vs USD | open.er-api | `https://open.er-api.com/v6/latest/USD` | `rates.AED`, `rates.EUR`, `rates.BRL`, `rates.ILS` |

Ambas se consultan en paralelo con `Promise.all` al cargar la página y en cada click del botón refresh.

---

## Lógica de conversión

Todas las conversiones pasan por **USD como moneda base**:

```js
// rates = { AED: X, USD: 1, EUR: ..., ARS: Y, BRL: Z, ILS: W }
//   (cada X = cuántas unidades de esa moneda equivalen a 1 USD)

al editar campo C:
  valorUSD = inputValue / rates[C]
  para cada otra moneda M:
    inputs[M].value = (valorUSD * rates[M]).toFixed(2)
```

---

## Modo offline (PWA)

La app funciona sin conexión gracias a tres capas:

### 1. Service Worker (`sw.js`)
Estrategia **stale-while-revalidate** para el app shell — sirve desde cache y revalida en background. Las APIs de tasas siempre van a la red (no se cachean datos volátiles).

Recursos cacheados al instalar:
- `index.html`, `manifest.json`
- Google Fonts (Fira Code, Fira Sans)
- Banderas SVG/PNG de cada moneda
- Logo MflowSuite

### 2. Manifest (`manifest.json`)
Permite **instalar** la app:
- Android/iOS: "Agregar a pantalla de inicio" → se abre como app standalone
- Desktop Chrome/Edge: ícono ⊕ en la barra de direcciones

### 3. Cache de tasas (`localStorage`)
La última consulta exitosa se guarda con timestamp. Si el fetch falla:
- Carga las tasas guardadas
- Muestra el cartel *"Offline · cache del DD/MM/AAAA HH:MM"*
- Permite seguir convirtiendo con esos valores

Si nunca hubo una consulta exitosa → error rojo pidiendo conexión.

---

## Estructura del proyecto

```
cambio-moneda/
├── index.html                                ← Aplicación (HTML + CSS + JS inline)
├── sw.js                                     ← Service Worker (offline app shell)
├── manifest.json                             ← Web App Manifest (PWA)
├── .gitignore
├── README.md
└── docs/superpowers/
    ├── specs/2026-05-11-cambio-moneda-design.md   ← Spec de diseño
    └── plans/2026-05-11-cambio-moneda.md          ← Plan de implementación
```

> Los archivos en `docs/` están en `.gitignore` — son artefactos internos del proceso de brainstorming/planning.

---

## Desarrollo local

```bash
# Clonar
git clone https://github.com/mflowsuite/cambio-moneda.git
cd cambio-moneda
```

**Importante:** el service worker **no funciona con `file://`** — requiere `http://` o `https://`. Para probarlo localmente:

```bash
# Con Python
python -m http.server 8000

# Con Node
npx serve .
```

Luego abrir [http://localhost:8000](http://localhost:8000).

### Probar el modo offline

1. Cargar la página al menos una vez (con internet)
2. DevTools → Application → Service Workers → confirmar que `sw.js` esté activo
3. DevTools → Network → Offline → recargar
4. La app sigue funcionando con las últimas tasas guardadas

---

## Deployment

El proyecto está deployado en GitHub Pages desde la rama `master`. Cualquier push a `master` se publica automáticamente en 1-2 minutos.

```bash
git add .
git commit -m "feat: descripción del cambio"
git push
```

> **Nota PWA:** cuando actualices `sw.js`, incrementá la constante `CACHE` (ej: `v2` → `v3`) para forzar a los browsers a descartar el cache viejo.

### Setup inicial (ya hecho)

```bash
gh repo create cambio-moneda --public --source=. --remote=origin --push
gh api repos/mflowsuite/cambio-moneda/pages \
  --method POST \
  -f "source[branch]=master" \
  -f "source[path]=/"
```

---

## Agregar una moneda nueva

1. **HTML** — duplicar una card existente y cambiar código ISO, nombre, bandera (`flagcdn.com/w40/<código-país>.png`)
2. **JS** — agregar la moneda en 5 lugares:
   - `const rates = { ..., XXX: null }`
   - `const CURRENCIES = [..., 'XXX']`
   - `rates.XXX = fxData.rates.XXX` en `fetchRates()`
   - `refs.XXX.textContent = ...` en `updateRateRefs()`
   - Guard en input handler: `if (!rates.XXX) return`
   - Guard en fallback offline del `refresh()` `catch`
   - Lista de monedas en `saveRatesCache()`
3. **sw.js** — agregar la URL de la bandera al array `SHELL` e incrementar la versión `CACHE`
4. **manifest.json** y **README** — actualizar la descripción si corresponde
5. Verificar que la moneda esté en `open.er-api.com/v6/latest/USD`

---

## Manejo de errores

| Situación | Comportamiento |
|-----------|---------------|
| Fetch OK | Actualiza tasas + guarda en cache |
| Fetch falla, hay cache | Usa cache + leyenda "Offline · cache del…" |
| Fetch falla, sin cache | Mensaje rojo, inputs deshabilitados |
| API devuelve dato inválido | Throw + mismo flujo de fallback |

Sin reintentos automáticos — el usuario usa el botón "Actualizar tasas".

---

## Decisiones de diseño

| Decisión | Por qué |
|----------|---------|
| Single-file HTML | YAGNI — no necesita build, fácil de mantener |
| USD como moneda base interna | Simplifica de O(n²) tasas cruzadas a O(n) |
| Botón refresh manual (no polling) | Las tasas no cambian segundo a segundo — preserva quota de las APIs |
| SW no cachea respuestas de APIs | Las tasas son datos volátiles — preferimos `localStorage` con TTL controlado |
| `localStorage` en vez de IndexedDB | Solo guardamos 6 floats — overkill cualquier cosa más compleja |
| Inputs `type="number"` con `inputmode="decimal"` | Teclado numérico en mobile + validación nativa |
| Dólar **blue venta** (no oficial) | Es el valor real de mercado en Argentina |
| Stale-while-revalidate (no network-first) | Carga instantánea desde cache + actualización en background |

---

## Powered by

Diseñado con el design system de **[MflowSuite](https://mflowsuite.com/)** — la marca de Martín Bauni.

---

## Licencia

MIT — usar, modificar y redistribuir libremente.
