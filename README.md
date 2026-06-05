# cambio-moneda

Convertidor de monedas responsive en tiempo real entre **AED**, **USD**, **ARS** (dólar blue venta), **BRL** e **ILS**. Single-file vanilla HTML/CSS/JS, deployado en GitHub Pages.

🌐 **Live:** [mflowsuite.github.io/cambio-moneda](https://mflowsuite.github.io/cambio-moneda/)

---

## Características

- **Conversión bidireccional en tiempo real** — al editar cualquier campo, los otros 4 se actualizan instantáneamente
- **Dólar blue venta** desde [Bluelytics](https://bluelytics.com.ar/) (sin API key)
- **AED, BRL e ILS** desde [open.er-api.com](https://www.exchangerate-api.com/docs/free) (sin API key)
- **Botón "Actualizar tasas"** con spinner, timestamp y manejo de errores
- **Responsive** — grilla 2×2 en tablet/desktop, 1 columna en mobile
- **Mobile-first** — `inputmode="decimal"` para teclado numérico nativo
- **Accesible** — `prefers-reduced-motion`, focus visible, sin scroll horizontal
- **Zero dependencias** — sin frameworks, sin build process, un solo `index.html`

---

## Stack

| Capa | Tecnología |
|------|------------|
| Frontend | HTML5 + CSS3 + Vanilla JS (ES2017+) |
| Tipografía | Fira Code · Fira Sans (Google Fonts) |
| Banderas | [flagcdn.com](https://flagcdn.com/) |
| Hosting | GitHub Pages |
| Design system | [MflowSuite](https://mflowsuite.com/) |

---

## APIs consumidas

| Dato | API | Endpoint | Campo |
|------|-----|----------|-------|
| Dólar blue venta | Bluelytics | `https://api.bluelytics.com.ar/v2/latest` | `blue.value_sell` |
| AED / BRL / ILS vs USD | open.er-api | `https://open.er-api.com/v6/latest/USD` | `rates.AED`, `rates.BRL`, `rates.ILS` |

Ambas se consultan en paralelo con `Promise.all` al cargar la página y en cada click del botón refresh.

---

## Lógica de conversión

Todas las conversiones pasan por **USD como moneda base**:

```js
// rates = { AED: X, USD: 1, ARS: Y, BRL: Z, ILS: W }
//   (cada X = cuántas unidades de esa moneda equivalen a 1 USD)

al editar campo C:
  valorUSD = inputValue / rates[C]
  para cada otra moneda M:
    inputs[M].value = (valorUSD * rates[M]).toFixed(2)
```

---

## Estructura del proyecto

```
cambio-moneda/
├── index.html                                ← Aplicación completa (HTML + CSS + JS inline)
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

# Abrir directamente en el browser — no requiere servidor
start index.html        # Windows
open index.html         # macOS
xdg-open index.html     # Linux
```

Para evitar limitaciones de CORS en algunos browsers al abrir desde `file://`, podés servirlo localmente:

```bash
# Con Python
python -m http.server 8000

# Con Node
npx serve .
```

---

## Deployment

El proyecto está deployado en GitHub Pages desde la rama `master`. Cualquier push a `master` se publica automáticamente en 1-2 minutos.

```bash
git add .
git commit -m "feat: descripción del cambio"
git push
```

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
2. **JS** — agregar la moneda en 4 lugares:
   - `const rates = { ..., XXX: null }`
   - `const CURRENCIES = [..., 'XXX']`
   - `rates.XXX = fxData.rates.XXX` en `fetchRates()`
   - `refs.XXX.textContent = ...` en `updateRateRefs()`
   - Guard en input handler: `if (!rates.XXX) return`
3. Verificar que la moneda esté en `open.er-api.com/v6/latest/USD`

---

## Manejo de errores

- Si alguna API falla → mensaje rojo inline debajo del botón
- Los inputs se deshabilitan hasta el próximo refresh exitoso
- Sin reintentos automáticos — el usuario usa el botón "Actualizar tasas"

---

## Decisiones de diseño

| Decisión | Por qué |
|----------|---------|
| Single-file HTML | YAGNI — no necesita build, fácil de mantener |
| USD como moneda base interna | Simplifica de O(n²) tasas cruzadas a O(n) |
| Botón refresh manual (no polling) | Las tasas no cambian segundo a segundo — preserva quota de las APIs |
| Inputs `type="number"` con `inputmode="decimal"` | Teclado numérico en mobile + validación nativa |
| Dólar **blue venta** (no oficial) | Es el valor real de mercado en Argentina |

---

## Powered by

Diseñado con el design system de **[MflowSuite](https://mflowsuite.com/)** — la marca de Martín Bauni.

---

## Licencia

MIT — usar, modificar y redistribuir libremente.
