# Divisor de Cuentas — guía para Claude

App de dividir cuentas grupales. **Archivo único `index.html`** (Alpine.js v3 + Pico.css v2 classless + Open Props), sin build step, desplegada en GitHub Pages.

- **App:** https://manfredengler.github.io/dividir_cuenta/
- **Docs:** https://manfredengler.github.io/dividir_cuenta/docs/ (MkDocs Material; fuente `docs-src/`, build comprometido en `docs/`)
- Repo GitHub: `manfredengler/dividir_cuenta` (ojo: la carpeta local se llama "dividir cuentas", con espacio — citar rutas siempre)

## Reglas duras del proyecto

1. **Todo en `index.html`.** No crear archivos JS/CSS aparte. No introducir build step ni framework.
2. **Privacidad:** los datos nunca salen del navegador. Nada de APIs externas para QR, analytics, etc. Compartir = estado codificado en la URL.
3. **El estado canónico es el contrato.** Campos nuevos: agregar default en `_cargarEstado()`, sumarlos a `persistir()`, `_urlEstado()` y al export/import Excel. Nunca renombrar campos existentes (rompe localStorage/URLs/Excels viejos).
4. **Dependencias nuevas: carga perezosa** (patrón de `_cargarXLSX()` / `mostrarQR()`). Jamás `<script src>` ni `<link rel="stylesheet">` bloqueante en `<head>` — el CSS de Pico+Open Props va inlineado desde v1.9.0 (Lighthouse perf 99; no introducir regresiones).
5. **Toda feature debe funcionar en ambas vistas:** tabla escritorio (≥768px) **y** editor móvil fullscreen (<768px). Controles duplicados comparten el mismo `x-model` — duplicar markup está bien, duplicar estado no.
6. **Accesibilidad no negociable:** WCAG AA (axe-core debe dar 0 violaciones), tap targets ≥44px en táctil, `aria-label` en todo control, anuncios vía `this._avisar()` (región `aria-live`).
7. **Español** en UI, código (nombres de funciones/variables), commits y docs.

## Flujo de trabajo establecido

1. Implementar en `index.html`.
2. **Verificar antes de publicar**: servidor local + Playwright (ver `.claude/verificacion.md`). El estado Alpine se manipula con `document.body._x_dataStack[0]` en `browser_evaluate`.
3. Documentar en el **changelog del `README.md`** (formato `### vX.Y.Z — Título` con checkboxes y evidencia de verificación).
4. Si cambia un contrato de datos o arquitectura → actualizar `docs-src/` y reconstruir: `uvx --with mkdocs-material mkdocs build`, commitear `docs/`.
5. Commit + push a `main` = deploy (GitHub Pages legacy, rama main, raíz). Tarda ~1 min; verificar con `curl` que el cambio llegó antes de re-auditar producción.

## Mapa rápido de index.html

| Zona | Qué hay |
|---|---|
| `<style>` | Tema (3 bloques: light/dark/auto), componentes, bloque táctil `@media (pointer: coarse), (max-width: 767px)`, view transitions |
| `<body>` | skip-link → header/navbar → main (secciones 01-06 autonumeradas por CSS counter) → footer → 3 `<dialog>` (drawer, editor ítem, QR) |
| `app()` | Estado arriba; luego: init/persistencia, tema, personas, ítems, asignación, compartir/QR/Excel, `_liquidar()`, `get resumen()` (motor de cálculo) |

Detalle completo en `docs-src/arquitectura.md` y `docs-src/formatos-de-datos.md`.

## Aprendizajes acumulados

Ver `.claude/aprendizajes.md` — gotchas reales encontrados durante el desarrollo (markers fantasma, cache del http.server, Web Share en Windows, etc.). Leerlo antes de tocar navbar, diálogos o el motor de cálculo.
