# Divisor de Cuentas

Divide cuentas grupales de forma equitativa. Archivo único `index.html`, sin servidor, sin build step. Funciona offline y se puede compartir por URL.

**[Abrir la app →](https://manfredengler.github.io/dividir_cuenta/)** · **[Documentación →](https://manfredengler.github.io/dividir_cuenta/docs/)**

---

## Stack

| Capa | Tecnología |
|---|---|
| Estructura | HTML5 semántico, WCAG AA |
| Estilos | Pico.css v2 (classless) + Open Props |
| Lógica | Alpine.js v3 |
| Persistencia | `localStorage` + URL hash (Base64 UTF-8) |
| Despliegue | GitHub Pages |

---

## Funcionalidades

### v0.1.0 — Cimiento
- [x] HTML5 semántico (`lang`, `viewport`, `color-scheme`)
- [x] CDN: Open Props + Pico.css v2 classless + Alpine.js v3
- [x] Dark mode automático (sistema operativo)
- [x] Switch de tema de 3 estados: automático → claro → oscuro (persistido, sin flash al recargar)
- [x] Identidad visual propia: tipografía Fraunces + Karla, paleta papel/tinta con acento esmeralda
- [x] Estado global con `x-data="app()"`
- [x] Selector de moneda (`$`, `€`, `S/.`, `£`, `R$`)
- [x] Toggle sin decimales

### v0.2.0 — Personas e Ítems
- [x] Agregar / eliminar participantes
- [x] Agregar / eliminar consumos (nombre + precio)
- [x] Al agregar ítem, todos los participantes quedan asignados por defecto
- [x] Al eliminar persona, se desvincula de todos los ítems
- [x] Labels vinculados a todos los inputs

### v0.3.0 — Motor de cálculos
- [x] Subtotal individual por consumo directo
- [x] Propina (%) distribuida proporcionalmente
- [x] Otros cargos fijos distribuidos proporcionalmente
- [x] Detección de ítems sin asignar con alerta `role="alert"`

### v0.4.0 — Interfaz adaptativa
- [x] Vista escritorio (`≥768px`): tabla matricial con checkboxes
- [x] Vista móvil (`<768px`): tarjetas por ítem con chips táctiles (`aria-pressed`)
- [x] Resumen por persona: consumo, propina, cargos, total, balance
- [x] Código de color en balances (positivo/negativo)

### v0.5.0 — Liquidación de deudas
- [x] Campo "Pagó al local" por persona
- [x] Marcador 💳 "paga la cuenta": esa persona cubre lo no declarado y recibe las transferencias
- [x] Tres métodos de liquidación seleccionables (persistidos y compartibles):
  - **Mínimas transferencias** — greedy, la menor cantidad de pagos posible
  - **Todo a una persona** — todos pagan al 💳 pagador (o mayor acreedor), que devuelve a los demás
  - **Proporcional** — cada deudor reparte entre los acreedores según lo que adelantó cada uno
- [x] Tabla de saldos individuales (debe pagar / pagó / balance / situación)
- [x] Cada transferencia muestra explícitamente quién paga a quién, con detalle del cálculo

### v0.7.0 — División avanzada por ítem
- [x] Modo "÷ igual": partes iguales entre los asignados (checkboxes / chips)
- [x] Modo "partes": fracciones ponderadas (ej. Ana 2, Beto 1 → 2/3 y 1/3)
- [x] Modo "%": porcentaje exacto por persona, con indicador de suma (verde al llegar a 100%)
- [x] El % sin asignar se reporta en la alerta de saldo pendiente

### v0.6.0 — Persistencia y compartición
- [x] Auto-guardado en `localStorage` con `x-effect`
- [x] Restauración desde URL hash (Base64 UTF-8) al cargar
- [x] Botón "Compartir": genera URL con estado codificado, copia al portapapeles
- [x] Anuncio accesible `aria-live` al copiar el enlace
- [x] Compatible con GitHub Pages (hash puro, sin servidor)

### v1.0.0 — Pulido
- [x] Formato monetario con `Intl.NumberFormat` (locale del navegador)
- [x] Landmarks semánticos (`<header>`, `<main>`, `<footer>`, `<section>`)
- [x] Open Props para spacing, radius y transiciones consistentes
- [ ] Auditoría accesibilidad con lector de pantalla
- [ ] Pruebas en Safari iOS / Chrome Android
- [ ] SRI hashes en las dependencias CDN

### v1.1.0 — Accesibilidad y PWA

Auditoría con axe-core (WCAG 2 AA) + revisión manual. Resultado previo: 1 violación seria, 40 reglas ok.

**Correcciones WCAG 2 AA:**
- [x] **Color contrast** (serious): `--pico-muted-color` claro pasó de `#8a8174` (3.56:1) a `#6b6459` (5.4:1) — afectaba el párrafo hero, los textos de estado vacío y el footer
- [x] **Skip-link**: enlace "Saltar al contenido" visible al recibir foco teclado (`.skip-link:focus`), apunta a `#contenido-principal`

**Mejoras de experiencia móvil:**
- [x] `inputmode="decimal"` en todos los campos numéricos de precio/partes/cargos — abre teclado numérico con coma decimal en iOS/Android
- [x] `inputmode="numeric"` en el campo de propina (entero)

**Metadatos y PWA:**
- [x] `<link rel="icon">` con SVG inline (÷ verde sobre fondo redondeado) — elimina el ícono genérico del navegador
- [x] `<meta name="theme-color">` para barra del navegador en móvil (verde en claro, oscuro en dark)
- [x] `<meta name="apple-mobile-web-app-title">` para el título al agregar a pantalla inicio en iOS
- [x] `crossorigin` en el `<link rel="preconnect">` de Google Fonts (faltaba en el primero)

**Resiliencia:**
- [x] `<noscript>` con aviso en rojo cuando JavaScript está deshabilitado

### v1.2.0 — Pagador por ítem
- [x] Selector "Pagó" en cada consumo: cualquier participante puede asociarse como quien pagó ese ítem
- [x] Disponible en la tabla de escritorio (columna "Pagó") y en las tarjetas móviles (fila "Pagado por")
- [x] El precio del ítem se suma a los pagos efectivos de esa persona y entra al cálculo de saldos y transferencias
- [x] Compatible con el campo manual "Pagó al local" y con el 💳 pagador global (que sigue cubriendo el resto no declarado)
- [x] Al eliminar una persona se desvincula como pagadora de sus ítems
- [x] Migración transparente de estados guardados antiguos (`pagadorItemId: null` por defecto)
- [x] Verificado E2E con Playwright: ítem de $100 pagado por Ana dividido entre 2 → transferencia "Beto → Ana $50"

### v1.3.0 — UX móvil (mobile-first)

Auditoría Lighthouse móvil (Performance 86 / A11y 100 / BP 100 / SEO 100) + inspección Playwright a 390×844 con datos sembrados. Se detectaron **28 controles por debajo del mínimo táctil** (botones eliminar de 26×26px, selects de 24px de alto, chips de 29px) y CSS render-blocking de fuentes (~880ms).

**Tap targets ≥ 44px (WCAG 2.5.5):**
- [x] Nuevo bloque `@media (pointer: coarse), (max-width: 767px)` con tamaños táctiles
- [x] Botones eliminar (×) y pagador (💳): de 26×26 a 44×44
- [x] Chips de asignación y métodos de liquidación: alto mínimo 44px
- [x] Selects de modo de división y "Pagado por": alto mínimo 40px
- [x] Botones e interruptor del header agrandados (el label completo es área táctil del switch)
- [x] Verificado con Playwright: 0 controles < 40px, sin overflow horizontal

**iOS:**
- [x] `font-size: 1rem` en inputs/selects en táctil — evita el zoom automático al enfocar campos

**Performance:**
- [x] Google Fonts con carga asíncrona (`media="print" onload`) — deja de bloquear el primer render (~880ms est.), con `<noscript>` de respaldo
- [x] Open Props + Pico.css combinados en un solo request (`cdn.jsdelivr.net/combine/...`) — un round-trip render-blocking menos
- [x] `preconnect` a `cdn.jsdelivr.net` (faltaba; solo existía para Google Fonts)

### v1.4.0 — Compartir nativo y navbar

**Compartir:**
- [x] En dispositivos táctiles, el botón Compartir abre el panel nativo del sistema (Web Share API) con WhatsApp, Facebook, Gmail, Telegram y todas las apps instaladas
- [x] En escritorio se mantiene el copiado al portapapeles (más directo que el diálogo de share de Windows/macOS)
- [x] Si el usuario cierra el panel no pasa nada; si el share falla por otra razón, cae al portapapeles
- [x] Ícono SVG de compartir en el botón
- [x] Verificado con Playwright simulando ambas ramas (`pointer: coarse` y escritorio)

**Navbar:**
- [x] En pantallas <560px la marca pasa arriba centrada y los controles se reparten en una fila completa debajo (antes quedaban apretados con saltos de línea irregulares)

### v1.4.1 — Botón de tema reconocible
- [x] Glifos unicode (◐ ☀ ☾) reemplazados por íconos SVG nítidos: círculo mitad relleno (auto), sol (claro), luna (oscuro)
- [x] El botón dejó de ser transparente: ahora tiene fondo de acento suave, borde esmeralda y el ícono en color primario
- [x] Hover con inversión de color (fondo esmeralda, ícono blanco)
- [x] Verificados los tres estados en tema claro y oscuro con capturas Playwright

### v1.4.2 — Microcopy
- [x] El switch "Sin dec." pasa a llamarse "Decimales", con lógica invertida: activado = mostrar decimales (el estado guardado `sinDecimales` se mantiene compatible)

### v1.5.0 — QR, footer y navbar móvil
- [x] **Compartir con código QR**: botón en la navbar que abre un `<dialog>` con el QR del enlace (estado completo de la cuenta)
  - Generado 100% en el navegador con `qrcode-generator` (sin servicios externos — coherente con la promesa de privacidad)
  - La librería se carga perezosamente solo al pulsar el botón (no afecta la performance inicial)
  - Fondo blanco fijo para que escanee bien en tema oscuro; aviso si el estado es demasiado grande para un QR
- [x] **Footer** con crédito y enlace al portafolio ([manfredengler.github.io](https://manfredengler.github.io/))
- [x] **Navbar estática en móvil** (<560px): ya no acompaña el scroll, libera espacio vertical de pantalla; en escritorio sigue sticky con blur

### v1.6.0 — Exportar e importar Excel
- [x] Nueva sección "Exportar e importar" con botones Descargar / Importar Excel
- [x] **Exportación** a `.xlsx` con 4 hojas:
  - `Participantes`: nombre, pagó al local, quién paga la cuenta
  - `Consumos`: ítem, precio, modo de división, pagado por, y una columna por persona (✗ asignado, o número de partes/%)
  - `Config`: propina, cargos, moneda, decimales, método de liquidación
  - `Resumen`: consumo, propina, cargos, total, pagó y balance por persona (para compartir el resultado)
- [x] **Importación** desde un Excel exportado: reconstruye participantes, consumos (los 3 modos de división), pagadores por ítem y configuración; valida el archivo y avisa por `aria-live` cuántos registros cargó
- [x] SheetJS (`xlsx`) cargado perezosamente solo al usar la función — no afecta la performance inicial
- [x] Verificado round-trip con Playwright: exportar → vaciar estado → importar produce totales idénticos (con división igual, por partes y por %)

### v1.7.0 — Sidebar móvil y editor de ítems fullscreen

**Navbar → drawer lateral (<560px):**
- [x] La navbar móvil queda solo con la marca y un botón hamburguesa
- [x] Los controles (moneda, decimales, tema, QR, compartir) viven en un drawer lateral derecho (`<dialog>` nativo: focus trap y Esc gratis)
- [x] Mismo estado Alpine que los controles de escritorio — siempre sincronizados
- [x] Cierre tocando el fondo oscurecido o con el botón ✕

**Edición de consumos → modal fullscreen (móvil):**
- [x] Las tarjetas de ítem son ahora compactas: nombre, precio, resumen legible de la división ("÷ igual entre Ana, Beto · pagó Ana") y botón Editar
- [x] El botón Editar abre un modal a pantalla completa con toda la edición: modo de división, pagado por, chips de asignación, partes/%
- [x] Botón "Eliminar ítem" con estilo destructivo (rojo) separado de "Listo"

**CSS View Transitions:**
- [x] Apertura/cierre animados con `document.startViewTransition`: el drawer desliza desde la derecha, el editor desde abajo (`view-transition-name` dedicados)
- [x] Fallback transparente en navegadores sin soporte y animaciones desactivadas con `prefers-reduced-motion`
- [x] Verificado con Playwright a 390×844: hamburguesa visible, controles ocultos, drawer y editor abren, modal ocupa el viewport completo y los chips editan el estado real

### v1.7.1 — Fixes móvil
- [x] Eliminado el cuadrado blanco junto a la hamburguesa: era el `::marker` del `li` (`display: list-item` → `display: block; list-style: none`)
- [x] Sin scroll horizontal: las celdas de tabla en móvil permiten quiebre de línea y usan padding compacto (antes `white-space: nowrap` desbordaba la tabla de saldos)
- [x] Botón cerrar (✕) de todos los diálogos ampliado a 44×44px con ícono de 1.5rem, hover/focus visible con fondo de acento

### v1.8.0 — Documentación MkDocs y specs actualizadas
- [x] Sitio de documentación con **MkDocs Material** servido en el mismo GitHub Pages: [/dividir_cuenta/docs/](https://manfredengler.github.io/dividir_cuenta/docs/)
  - Fuente en `docs-src/`, build comprometido en `docs/` (sin workflow aparte: construir y commitear es el deploy)
  - Páginas: Inicio, Guía de uso, Arquitectura, Formatos de datos, Extensibilidad
  - `uvx --with mkdocs-material mkdocs build` para regenerar
- [x] `specs.md` actualizado al estado real: stack vigente, requisitos EARS nuevos (REQ-02b/c, 03b, 04b, 08b, 09b, 16), casos de uso de partes/%, pagador por ítem, round-trip Excel y compartir nativo
- [x] Enlace "Documentación" en el footer de la app y en este README (de paso se corrigió la URL de demo, que apuntaba a un repo inexistente)

### v1.8.1 — Fin del scroll horizontal en móvil

Diagnóstico con Playwright midiendo `getBoundingClientRect` de todo el DOM a 360px y 320px con contenido extremo (nombres largos, todas las secciones visibles). Tres culpables reales:

- [x] **Input file oculto (Importar Excel)**: los selectores `input:not(...)` de Pico superan en especificidad a `.sr-only` y lo estiraban a ancho completo → `.sr-only` ahora usa `!important` en sus propiedades críticas
- [x] **Tarjetas de ítem**: con nombres largos el contenedor del nombre no se encogía (`min-width: auto` de flex) y empujaba los botones Editar/× fuera del viewport → `min-width: 0` + `overflow-wrap: anywhere` (también aplicado a `.persona-nombre`)
- [x] **Badge "💳 paga la cuenta"** en el resumen: el header de la tarjeta no quebraba con nombres largos → `flex-wrap: wrap`
- [x] Red de seguridad: `html { overflow-x: clip }` — ningún desborde accidental futuro genera scroll de página
- [x] Verificado a 360px y 320px: `scrollWidth === clientWidth`, cero elementos fuera del documento

### v1.8.2 — Botón "Invitame un café"
- [x] Badge de Buy Me a Coffee en el footer enlazando a [buymeacoffee.com/manfredengm](https://buymeacoffee.com/manfredengm)
- [x] Implementación segura: enlace estático + SVG inline propio — **sin el widget oficial ni imágenes de terceros** (cero scripts externos, cero requests por visita, coherente con la promesa de privacidad y sin impacto en performance)
- [x] `target="_blank"` con `rel="noopener noreferrer"`; tap target 164×44px verificado

### v1.8.3 — Fila de participante legible en móvil
- [x] **Bug**: en viewports angostos el nombre del participante se partía letra por letra — los controles de ancho fijo de la fila (input "Pagó", 💳, ×, avatar) consumían casi todo el ancho y el nombre (`flex: 1` + `overflow-wrap: anywhere`) quedaba con ~1 carácter
- [x] **Fix**: bajo 768px la `.persona-row` pasa a grilla de 2 líneas — nombre arriba a ancho completo, controles abajo (input flexible + botones alineados a la derecha); el avatar abarca ambas líneas. Escritorio sin cambios (sigue flex en una línea)
- [x] Verificado con Playwright a 390px y 320px con nombre extremo ("María Fernanda de los Ángeles"): nombre en una sola línea (263px de ancho a 390px), tap targets 44×44px, cero overflow horizontal, escritorio intacto

### v1.9.0 — Performance: CSS inlineado y CLS ~0
Ataque a las dos causas reales del Lighthouse Perf 88 (móvil):
- [x] **CSS bloqueante eliminado**: Open Props + Pico (el combine de jsDelivr, ~940 ms de FCP/LCP estimados) ahora va inlineado en un `<style>` del `<head>` — coherente con la filosofía de archivo único; el gzip de GitHub Pages absorbe los ~19 KB extra. La URL fuente queda comentada en el HTML para futuras actualizaciones
- [x] **CLS 0.201 → ~0**: el `x-cloak` de `<main>` ocultaba todo el contenido con `display: none` hasta que Alpine arrancaba y el footer saltaba ~1300px. Ahora `main[x-cloak]` usa `visibility: hidden` con `display: block`, reservando el espacio
- [x] Verificado con Playwright: variables de Pico/Open Props resueltas, cero stylesheets de jsDelivr, motor de cálculo intacto (caso de referencia Beto → Ana $50), axe-core 0 violaciones
- [x] Lighthouse móvil contra producción tras el deploy: **Perf 99 / A11y 100 / BP 100 / SEO 100** (antes Perf 88) — FCP 1.1s (antes 2.0s), CLS 0.002 (antes 0.201), TBT 0ms

---

## Uso local

Abre `index.html` directamente en el navegador. No requiere servidor.

```
open index.html        # macOS
start index.html       # Windows
xdg-open index.html    # Linux
```

## Despliegue en GitHub Pages

1. Sube el repositorio a GitHub
2. Settings → Pages → Branch: `main` / Folder: `/ (root)`
3. La app queda disponible en `https://<usuario>.github.io/<repo>/`
