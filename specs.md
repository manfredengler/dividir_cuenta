A continuación, se presentan la definición del proyecto, las especificaciones técnicas en formato **EARS**, los casos de uso en formato **Given-When-Then**, y el estado de implementación.

> **Documentación extendida:** la guía de uso, la arquitectura y la guía de extensibilidad viven en [la documentación MkDocs](https://manfredengler.github.io/dividir_cuenta/docs/) (`docs-src/` en este repo). El historial detallado de cambios está en el [README](README.md).

---

# Definición y Resumen del Proyecto

**Divisor de Cuentas** — https://manfredengler.github.io/dividir_cuenta/

Archivo único `index.html` ejecutable localmente y desplegado en **GitHub Pages** (sin servidor, sin build step). Facilita la división detallada de consumos grupales usando **Alpine.js v3** para reactividad y **Pico.css v2** para diseño adaptativo minimalista con soporte nativo de modo oscuro/claro.

### Principios de Diseño
- **Minimalista:** Solo lo necesario. Sin abstracciones prematuras ni features de adorno.
- **Moderno:** HTML semántico, APIs nativas del browser (Clipboard, Web Share, View Transitions, `<dialog>`, Intl, localStorage).
- **Accesible:** Landmarks semánticos, labels vinculados, contraste WCAG AA, tap targets ≥44px, skip-link, anuncios `aria-live`.
- **Privado:** Los datos nunca salen del navegador. El QR y el Excel se generan localmente; compartir codifica el estado en la propia URL.
- **Sin dependencias locales:** Todo vía CDN; las librerías pesadas (QR, SheetJS) se cargan perezosamente solo al usarse.

### Stack Técnico
| Capa | Tecnología | Notas |
|---|---|---|
| Estructura | HTML5 semántico | `lang="es"`, landmarks, `<dialog>` nativo |
| Estilos | Pico.css v2 classless + Open Props (CDN combinado) | Dark mode nativo; tipografía Fraunces + Karla (carga asíncrona) |
| Lógica | Alpine.js v3 (CDN, defer) | `x-data`, `x-effect`, getter reactivo `resumen` |
| Persistencia | `localStorage` + URL hash (Base64 UTF-8) | Sin backend |
| Compartir | Clipboard API, Web Share API (táctil), QR local (`qrcode-generator`, lazy) | |
| Import/Export | SheetJS `xlsx` (lazy) | 4 hojas: Participantes, Consumos, Config, Resumen |
| Animación | CSS View Transitions (con fallback y `prefers-reduced-motion`) | Drawer y editor móvil |
| Despliegue | GitHub Pages (rama `main`, raíz) | Docs MkDocs pre-compiladas en `/docs` |

---

# 1. Especificaciones de Requisitos (Formato EARS)

### Plataforma y Despliegue
- **REQ-00a (Ubiquitous):** El sistema debe funcionar como archivo único `index.html` sin proceso de build, servido por GitHub Pages desde la rama `main`.
- **REQ-00b (Ubiquitous):** El sistema debe declarar `viewport`, `lang="es"`, `color-scheme`, `theme-color` (por tema) y favicon SVG inline.
- **REQ-00c (Ubiquitous):** Las librerías no críticas (Google Fonts, QR, SheetJS) no deben bloquear el primer render: fuentes asíncronas y scripts de carga perezosa bajo demanda.

### Gestión de Datos y Estado
- **REQ-01 (Ubiquitous):** El sistema debe almacenar el estado (personas, ítems con modo de división/partes/pagador, cargos, configuración, método de liquidación) en `localStorage` ante cualquier cambio usando `x-effect`.
- **REQ-02 (Event-driven):** CUANDO el usuario active "Compartir" en escritorio, EL sistema debe codificar el estado completo en Base64 UTF-8, construir la URL con hash y copiarla al portapapeles con confirmación inline y anuncio `aria-live`.
- **REQ-02b (Event-driven):** CUANDO el usuario active "Compartir" en un dispositivo táctil (`pointer: coarse`), EL sistema debe abrir el panel nativo de compartir (Web Share API) con la URL; si el usuario lo cancela no debe ocurrir nada; si falla debe caer al portapapeles.
- **REQ-02c (Event-driven):** CUANDO el usuario active "Código QR", EL sistema debe generar localmente (sin servicios externos) un QR de la URL con estado y mostrarlo en un `<dialog>` con fondo blanco fijo.
- **REQ-03 (Event-driven):** CUANDO la app cargue y `location.hash` contenga datos, EL sistema debe decodificar el Base64 e inicializar el estado ignorando `localStorage`.
- **REQ-03b (Event-driven):** CUANDO el usuario exporte a Excel, EL sistema debe generar un `.xlsx` con hojas `Participantes`, `Consumos`, `Config` y `Resumen`; CUANDO importe un archivo así exportado, EL sistema debe reconstruir el estado completo (incluidos los tres modos de división y pagadores por ítem) o avisar si el archivo no es válido.

### Cálculos y Distribución
- **REQ-04 (Ubiquitous):** El sistema debe dividir cada ítem según su modo: **igual** (equitativo entre asignados), **partes** (fracciones ponderadas) o **%** (porcentaje exacto por persona).
- **REQ-04b (Event-driven):** CUANDO un ítem tenga asociado un pagador ("Pagado por"), EL sistema debe sumar su precio a los pagos efectivos de esa persona.
- **REQ-05 (Ubiquitous):** El sistema debe distribuir propina (%) y cargos fijos proporcionalmente al subtotal individual.
- **REQ-06 (Unwanted Behavior):** SI un ítem no tiene asignación (o el % suma menos de 100), EL sistema debe excluir el monto no asignado del total y mostrar alerta `role="alert"` con el pendiente.

### Formato e Interfaz
- **REQ-07 (State-driven):** MIENTRAS el switch "Decimales" esté desactivado, EL sistema debe redondear los valores al entero.
- **REQ-08 (State-driven):** MIENTRAS el viewport sea <768px, EL sistema debe mostrar los ítems como tarjetas compactas (nombre, precio, resumen de división) cuya edición ocurre en un modal a pantalla completa; ≥768px debe mostrar la tabla matricial.
- **REQ-08b (State-driven):** MIENTRAS el viewport sea <560px, EL sistema debe colapsar los controles de la navbar en un drawer lateral accesible mediante botón hamburguesa, y el header no debe ser sticky.
- **REQ-09 (Ubiquitous):** El sistema debe formatear montos con `Intl.NumberFormat` (locale del navegador) y el símbolo de moneda elegido.
- **REQ-09b (Ubiquitous):** Las aperturas/cierres de drawer y editor deben animarse con CSS View Transitions cuando exista soporte, sin animación como fallback y respetando `prefers-reduced-motion`.

### Liquidación de Deudas
- **REQ-10 (Ubiquitous):** El sistema debe calcular el balance por persona: `total_a_pagar − pagos_efectivos` (pagos declarados + ítems pagados + resto cubierto por el 💳 pagador global).
- **REQ-11 (Event-driven):** CUANDO existan saldos, EL sistema debe generar el plan de transferencias según el método elegido: **mínimas transferencias** (greedy), **todo a una persona** (hub) o **proporcional**, cada una con detalle del cálculo.

### Accesibilidad
- **REQ-12 (Ubiquitous):** Landmarks semánticos, skip-link visible con foco, y región `aria-live` para anuncios.
- **REQ-13 (Ubiquitous):** Todos los inputs/selects con `<label>` o `aria-label`; botones con nombre accesible.
- **REQ-14 (Ubiquitous):** Chips y toggles operables por teclado con `aria-pressed`; diálogos con `<dialog>` nativo (focus trap, Esc).
- **REQ-15 (Ubiquitous):** Contraste WCAG AA (≥4.5:1) en ambos temas (auditado con axe-core: 0 violaciones).
- **REQ-16 (State-driven):** MIENTRAS el dispositivo sea táctil o el viewport <768px, los controles interactivos deben medir ≥44×44px y los inputs usar fuente ≥16px (evita zoom iOS) e `inputmode` numérico apropiado.

---

# 2. Casos de Uso y Pruebas (Formato Given-When-Then)

### Caso 1: División de ítem con propina proporcional
- **GIVEN:** Ana y Beto. Ítem "Pizza" $10.000 asignado a ambos. Propina 10%.
- **WHEN:** El sistema procesa totales.
- **THEN:** Subtotal $5.000 c/u, propina $500 c/u, total individual $5.500 c/u.

### Caso 2: Ítem sin asignar — alerta accesible
- **GIVEN:** "Bebida" $2.000 sin asignar.
- **WHEN:** El sistema calcula.
- **THEN:** Alerta `role="alert"` con el monto pendiente; el total excluye $2.000.

### Caso 3: División por partes y porcentaje
- **GIVEN:** "Vino" $60 en modo partes (Ana 2, Beto 1); "Postre" $40 en modo % (Ana 25, Beto 75).
- **WHEN:** El sistema calcula.
- **THEN:** Vino: Ana $40, Beto $20. Postre: Ana $10, Beto $30. Si el % sumara <100, el resto aparece como pendiente.

### Caso 4: Pagador por ítem y liquidación
- **GIVEN:** "Pizza" $100 dividida entre Ana y Beto, con "Pagado por: Ana".
- **WHEN:** El sistema liquida (método mínimas transferencias).
- **THEN:** Pagos efectivos Ana $100. Transferencia única: "Beto ⟶ Ana $50".

### Caso 5: Importación de estado por URL
- **GIVEN:** URL con hash Base64 de un estado.
- **WHEN:** Alpine.js inicializa.
- **THEN:** El estado de la URL reemplaza al de `localStorage`.

### Caso 6: Round-trip Excel
- **GIVEN:** Estado con los 3 modos de división, pagadores por ítem y configuración.
- **WHEN:** Se exporta a `.xlsx`, se vacía el estado y se importa el archivo.
- **THEN:** Los totales por persona son idénticos a los previos a la exportación.

### Caso 7: Compartir nativo en móvil
- **GIVEN:** Dispositivo con `pointer: coarse` y soporte de Web Share API.
- **WHEN:** El usuario pulsa "Compartir".
- **THEN:** Se abre el panel del sistema (WhatsApp, Gmail, etc.) con la URL; en escritorio se copia al portapapeles.

### Caso 8: Modo oscuro de 3 estados
- **GIVEN:** Usuario con SO en modo oscuro y tema de la app en "auto".
- **WHEN:** La página carga.
- **THEN:** Interfaz oscura sin flash (script en `<head>` aplica el tema persistido antes del primer render). El botón cicla auto → claro → oscuro.

---

# 3. Estado de Implementación

Todas las fases del plan original (v0.1.0 → v1.0.0) están **completas**, más las extensiones v1.1.0 → v1.7.1 (accesibilidad auditada, UX móvil táctil, pagador por ítem, Web Share + QR, Excel, drawer lateral, editor fullscreen con View Transitions).

El changelog detallado y verificable (con la evidencia de cada auditoría Lighthouse/axe/Playwright) se mantiene en el **[README](README.md)**. Las decisiones de arquitectura y la guía para extender el sistema están en la **[documentación](https://manfredengler.github.io/dividir_cuenta/docs/)**.
