# Flujo de verificación (antes de cada push)

Receta usada en todas las features publicadas. Adaptar según lo tocado.

## 1. Servidor local

```bash
# en background; matar al terminar
python -m http.server 8742
```

Navegar con Playwright MCP a `http://localhost:8742/?v=N` — **incrementar N en cada edición** (cache-bust; Chrome cachea el HTML aunque http.server no mande headers).

## 2. Sembrar estado de prueba

En `browser_evaluate`, el componente Alpine completo está en `document.body._x_dataStack[0]`:

```js
const root = document.body._x_dataStack[0];
root.personas = [{id:1,nombre:'Ana',pagado:0},{id:2,nombre:'Beto',pagado:0}];
root.items = [{id:1,nombre:'Pizza',precio:100,modo:'igual',partes:{},pagadorItemId:1,asignados:[1,2]}];
// esperar ~200ms a que Alpine re-renderice antes de medir el DOM
```

Caso de referencia: Pizza $100 pagada por Ana entre ambos ⇒ transferencia "Beto → Ana $50".

## 3. Chequeos según lo tocado

| Cambio | Verificar |
|---|---|
| UI móvil | Viewport 390×844: tap targets ≥40px, `document.documentElement.scrollWidth - innerWidth === 0` (sin overflow) |
| Accesibilidad | Inyectar axe-core desde CDN y `axe.run()` → 0 violations |
| Cálculo | Comparar `root.resumen.totales/saldos/transferencias` contra valores a mano |
| Compartir | Stubear `navigator.share` y `matchMedia` para probar ambas ramas (táctil/escritorio) |
| Excel | Round-trip: `_wbExportar()` → `XLSX.write(type:'array')` → `new File` → `importarExcel({target:{files:[f],value:''}})` → totales idénticos |
| Tema | `root.tema = 'light'|'dark'|'auto'; root._aplicarTema()` + captura del header en ambos |

## 4. Lighthouse (si se tocó performance o head)

```bash
npx -y lighthouse https://manfredengler.github.io/dividir_cuenta/ \
  --output=json --output-path=./lh.json \
  --form-factor=mobile --screenEmulation.mobile \
  --chrome-flags="--headless=new" --quiet
# parsear lh.json con python; borrar el json al final (está en .gitignore)
```

Línea base actual: **Perf 99 / A11y 100 / BP 100 / SEO 100**. No publicar regresiones.

## 5. Publicar

1. Changelog en `README.md` (`### vX.Y.Z — Título`, checkboxes, evidencia).
2. Si cambió arquitectura o contratos: `docs-src/` + `uvx --with mkdocs-material mkdocs build` (regenera `docs/`).
3. Commit en español + push a `main`.
4. Esperar el deploy (~30-90s): `until curl -s <url> | grep -q "<string nueva>"; do sleep 5; done`.
5. Si era cambio de performance: re-correr Lighthouse contra producción.
