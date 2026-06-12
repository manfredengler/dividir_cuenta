# Divisor de Cuentas

Divide cuentas grupales de forma equitativa. Archivo único `index.html`, sin servidor, sin build step. Funciona offline y se puede compartir por URL.

**[Ver demo →](https://manfred-engler.github.io/dividir-cuentas/)**

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
