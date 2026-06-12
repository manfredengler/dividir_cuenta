# Arquitectura

Toda la aplicación vive en un único `index.html` (~2.000 líneas) sin build step. La estructura interna, de arriba hacia abajo:

```
index.html
├── <head>
│   ├── Metadatos (viewport, color-scheme, theme-color, favicon SVG inline)
│   ├── CSS de CDN: Open Props + Pico.css v2 classless (un solo request /combine/)
│   ├── Google Fonts (Fraunces + Karla) con carga asíncrona
│   ├── <style> propio: tema, componentes, responsive, view transitions
│   └── Script anti-flash: aplica el tema persistido antes del primer render
├── <body x-data="app()" x-init="init()" x-effect="persistir()">
│   ├── Skip-link + región aria-live de anuncios
│   ├── <header>: navbar (drawer en móvil)
│   ├── <main>: secciones 01-06
│   ├── <footer>
│   ├── <dialog> × 3: drawer, editor de ítem, QR
│   └── <script> función app() — todo el estado y la lógica
└── Alpine.js v3 (defer, CDN)
```

## Capa de estado (Alpine.js)

Un único componente `app()` en `<body>` contiene todo. Estado principal:

```js
personas: [{ id, nombre, pagado }]
items: [{ id, nombre, precio, modo, partes: {pid: n}, asignados: [pid], pagadorItemId }]
propinaPct, cargosExtra        // cargos
moneda, sinDecimales, tema     // preferencias
pagadorId                      // 💳 pagador global
metodoLiquidacion              // 'minimo' | 'central' | 'proporcional'
```

Tres mecanismos reactivos hacen todo el trabajo:

1. **`x-effect="persistir()"`** — cualquier mutación de estado se serializa a `localStorage` automáticamente. No hay llamadas manuales a "guardar".
2. **Getter `get resumen()`** — el motor de cálculo completo es un getter computado: subtotales, propina/cargos proporcionales, pagos efectivos, saldos y transferencias se recalculan ante cualquier cambio. La UI solo *lee* `resumen.*`.
3. **`init()`** — al cargar: si hay `location.hash`, decodifica el estado de la URL (prioridad); si no, restaura `localStorage`.

## Motor de cálculo

`get resumen()` ejecuta en orden:

1. **Subtotales por persona** — itera ítems aplicando su modo (`igual` / `partes` / `pct`). Lo no asignado se acumula como "huérfano" y se excluye.
2. **Propina y cargos** — distribuidos en proporción al subtotal de cada quien.
3. **Pagos efectivos** — `p.pagado` declarado + suma de ítems con `pagadorItemId = p.id` + el resto no declarado si `p.id === pagadorId`.
4. **Saldos** — `total − pagosEfectivos` por persona.
5. **Transferencias** — `_liquidar(saldos)` despacha según `metodoLiquidacion` (greedy / hub / proporcional).

Tolerancia numérica: los comparadores usan `0.005` como épsilon para evitar transferencias de centavos fantasma por flotantes.

## Capa de UI

- **Responsive por CSS, no por JS**: la tabla matricial (`.vista-tabla`, ≥768px) y las tarjetas (`.vista-cards`, <768px) existen ambas en el DOM; los media queries deciden cuál se ve.
- **Diálogos nativos `<dialog>`**: drawer de menú, editor de ítem fullscreen y QR usan `showModal()` — focus trap, Esc y backdrop vienen gratis del navegador.
- **View Transitions**: `_conTransicion(fn)` envuelve apertura/cierre con `document.startViewTransition` si existe y `prefers-reduced-motion` no está activo; si no, ejecuta `fn` directo. Las animaciones (slide lateral del drawer, slide vertical del editor) se definen en CSS sobre `::view-transition-new/old(nombre)`.
- **Tema de 3 estados**: `auto | light | dark` persistido en `localStorage` aparte (`dc-tema`) para poder aplicarlo con un script síncrono en `<head>` antes del primer paint (sin flash).

## Carga perezosa

Las dependencias pesadas no están en el HTML inicial; se inyectan como `<script>` la primera vez que se usan:

| Librería | Se carga al… | Función |
|---|---|---|
| `qrcode-generator` | pulsar "Código QR" | `mostrarQR()` |
| SheetJS (`xlsx`) | exportar o importar Excel | `_cargarXLSX()` |

Ambas con manejo de error (anuncio `aria-live` si el CDN falla).

## Métricas (Lighthouse móvil)

Performance **86** · Accesibilidad **100** · Best Practices **100** · SEO **100**. El FCP está dominado por el único CSS bloqueante restante (Pico); inlinearlo es el siguiente paso natural si se quiere subir Performance.
