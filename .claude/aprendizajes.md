# Aprendizajes del proyecto (gotchas reales)

Cosas que ya nos mordieron una vez. Leer antes de tocar el área correspondiente.

## CSS / Layout

- **`display: list-item` dibuja el `::marker`** aunque el `ul` padre tenga `list-style: none`. Al mostrar/ocultar `li` por media query usar `display: block; list-style: none;` (bug del "cuadrado blanco" junto a la hamburguesa, v1.7.1).
- **`white-space: nowrap` en celdas desborda en móvil.** Las tablas (saldos, matricial) necesitan `white-space: normal` + padding compacto bajo 560px. El overflow queda contenido en el `figure` con `overflow-x: auto`, pero el scrollbar se ve feo igual.
- **Especificidad Pico:** `header nav button` (3 elementos) le gana a `.btn-tema` (1 clase). Si un override no aplica, revisar especificidad contra los selectores de elemento de Pico antes de sospechar de la cascada.
- **El botón cerrar de Pico (`button[rel=prev]`) es diminuto** — ya está corregido globalmente a 44×44px; los diálogos nuevos lo heredan gratis si usan `<button rel="prev">` en el header del `article`.
- **Contraste:** `--pico-muted-color` se sobreescribe en los TRES bloques de tema (light, dark, y el `@media prefers-color-scheme` para auto). Cambiar un color de tema = tocar los 3 lugares.
- **Scroll horizontal en móvil — los 3 patrones que lo causaron (v1.8.1):**
  1. `.sr-only` sin `!important` pierde contra `input:not(...)` de Pico → el input file oculto quedaba a ancho completo.
  2. Hijos de flex no se encogen por defecto (`min-width: auto`): textos largos empujan botones fuera del viewport. Antídoto: `min-width: 0` + `overflow-wrap: anywhere` en el hijo que contiene texto.
  3. Headers flex sin `flex-wrap: wrap` con badges `nowrap`.
  Para diagnosticar: iterar `getBoundingClientRect()` de todo el DOM filtrando los que exceden `clientWidth` y no están dentro de un ancestro con `overflow-x` scroll/auto/clip. Probar a 320px con nombres largos. Hay `html { overflow-x: clip }` como red de seguridad, pero arreglar siempre la causa raíz.

## Alpine.js

- **`x-cloak` solo en `<main>`**: los SVGs condicionales del header (tema) necesitan `x-cloak` individual en los que no son el default, o se ven los tres durante el primer paint.
- **Selects con valor null:** `x-model.number` no maneja bien `null`. Patrón usado: `:value="campo ?? ''"` + `@change="campo = $event.target.value === '' ? null : Number($event.target.value)"`.
- **`x-if` dentro de `<dialog>`** funciona bien para contenido pesado (editor de ítem): el template solo se monta cuando `itemEdit` existe; `@close` del dialog lo limpia.
- **Acceso al estado desde tests:** `document.body._x_dataStack[0]` expone el componente completo en `browser_evaluate`. Tras mutar arrays, dar ~200ms para que Alpine re-renderice antes de medir el DOM.

## APIs del navegador

- **Web Share API existe en Chrome de Windows** — no alcanza `navigator.share` como detección de móvil. Se gatea con `matchMedia('(pointer: coarse)').matches` para que escritorio copie al portapapeles.
- **`navigator.share` con AbortError = usuario canceló**: no hacer fallback en ese caso (se duplicaría la acción); solo caer a portapapeles en otros errores.
- **View Transitions:** envolver siempre en `_conTransicion()`, que ya chequea soporte + `prefers-reduced-motion`. Las animaciones van en CSS sobre `::view-transition-new/old(nombre)`; el `view-transition-name` se asigna al dialog.
- **iOS hace zoom al enfocar inputs con fuente <16px** — el bloque táctil ya fuerza `font-size: 1rem`; no crear inputs fuera de ese alcance.
- **El QR necesita fondo blanco fijo** (no variable de tema) para escanearse en modo oscuro.

## Verificación / Tooling

- **`python -m http.server` no manda headers de cache pero Chrome cachea igual** el HTML por heurística. Tras editar, navegar con query de cache-bust (`?v=N`) o las pruebas dan falsos resultados contra la versión vieja (nos pasó verificando el gate de Web Share).
- **Lighthouse en Windows:** escribir el reporte al cwd (`--output-path=./lh.json`), no a `/tmp` (el bash y el python ven distintos /tmp). El EPERM al final del run es solo cleanup del perfil de Chrome; el JSON ya está escrito.
- **GitHub Pages tarda ~30-90s** en servir un push. Verificar con `curl ... | grep <string nueva>` en loop antes de re-auditar producción.
- **jsDelivr `/combine/`** une varios paquetes en un request: `https://cdn.jsdelivr.net/combine/npm/a,npm/b`. Usado para Open Props + Pico.
- **`gh api repos/.../pages --method POST`** necesita `--input -` con JSON (el `--field source='{...}'` manda string, no objeto).

## Motor de cálculo

- **Épsilon 0.005 en todas las comparaciones de saldos** — evita transferencias fantasma de centavos por flotantes. Mantenerlo en cualquier lógica nueva de dinero.
- **El % que no suma 100 va a "huérfanos"**: el resto no asignado se reporta en la alerta, no se reparte en silencio. Cualquier modo nuevo debe seguir esa filosofía.
- **`pagosEfectivos` tiene 3 fuentes** que se suman: `p.pagado` declarado + ítems con `pagadorItemId` + resto cubierto por el `pagadorId` global. Tocar una requiere revisar `declarado`/`restante` para no contar doble.

## Pendientes conocidos (ideas a futuro)

- Performance 86: el paso siguiente es **inlinear el CSS de Pico+Open Props** en el HTML (~19KB, coherente con filosofía de archivo único, pierde caché CDN).
- SRI hashes en los CDN (pendiente del spec original; complicado con `@1`/`@2` flotantes — requeriría pinear versiones exactas).
- Pruebas reales en Safari iOS / Chrome Android (solo se verificó con emulación).
- PWA completa (manifest + service worker) para instalar y usar offline real.
