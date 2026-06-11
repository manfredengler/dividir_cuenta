A continuación, se presentan la definición del proyecto, las especificaciones técnicas en formato **EARS**, los casos de uso en formato **Given-When-Then**, y el plan de implementación como **Changelog de Checkpoints**.

---

# Definición y Resumen del Proyecto

**Divisor de Cuentas**
Archivo único `index.html` ejecutable localmente y desplegado en **GitHub Pages** (sin servidor, sin build step). Facilita la división detallada de consumos grupales usando **Alpine.js v3** para reactividad y **Pico.css v2** para diseño adaptativo minimalista con soporte nativo de modo oscuro/claro.

### Principios de Diseño
- **Minimalista:** Solo lo necesario. Sin abstracciones prematuras ni features de adorno.
- **Moderno:** HTML semántico, APIs nativas del browser (Clipboard, URLSearchParams, Intl, localStorage).
- **Accesible:** Landmarks semánticos, jerarquía de encabezados, labels vinculados, contraste WCAG AA.
- **Sin dependencias locales:** Todo vía CDN con integridad verificada (SRI hashes).

### Stack Técnico
| Capa | Tecnología | Notas |
|---|---|---|
| Estructura | HTML5 semántico | `<!DOCTYPE html>`, `lang="es"`, landmarks |
| Estilos | Pico.css v2 + Open Props (CDN + SRI) | Dark mode nativo; Open Props aporta tokens de color, spacing y animaciones |
| Lógica | Alpine.js v3 (CDN + SRI) | `x-data`, `x-effect`, sin build step |
| Persistencia | `localStorage` + URL hash (Base64) | Sin backend |
| Despliegue | GitHub Pages (rama `main`, carpeta raíz) | URL base estática predecible |

---

# 1. Especificaciones de Requisitos (Formato EARS)

### Plataforma y Despliegue
- **REQ-00a (Ubiquitous):** El sistema debe funcionar como archivo único `index.html` sin proceso de build, servido directamente por GitHub Pages desde la rama `main`.
- **REQ-00b (Ubiquitous):** El sistema debe declarar `<meta name="viewport" content="width=device-width, initial-scale=1.0">` y `<html lang="es">` para garantizar responsividad y soporte de lectores de pantalla.
- **REQ-00c (Ubiquitous):** Todas las dependencias CDN (Pico.css, Open Props, Alpine.js) deben incluir atributo `integrity` (SRI hash) y `crossorigin="anonymous"` para verificación criptográfica.
- **REQ-00d (Ubiquitous):** El sistema debe declarar `<meta name="color-scheme" content="light dark">` para que los controles nativos del browser (inputs, scrollbars) adopten el tema del sistema operativo automáticamente.

### Gestión de Datos y Estado
- **REQ-01 (Ubiquitous):** El sistema debe almacenar el estado (personas, ítems, cargos, configuración) en `localStorage` ante cualquier cambio usando `x-effect` de Alpine.js.
- **REQ-02 (Event-driven):** CUANDO el usuario active "Copiar Enlace", EL sistema debe codificar el estado completo en Base64, construir la URL con hash (`location.hash`) y copiarla al portapapeles via `navigator.clipboard.writeText()`, mostrando confirmación inline breve.
- **REQ-03 (Event-driven):** CUANDO la app cargue y `location.hash` contenga datos, EL sistema debe decodificar el Base64 e inicializar el estado ignorando `localStorage`.

### Cálculos y Distribución
- **REQ-04 (Ubiquitous):** El sistema debe dividir el precio de cada ítem equitativamente entre los comensales asignados a dicho ítem.
- **REQ-05 (Ubiquitous):** El sistema debe distribuir propina (%) y cargos fijos de forma proporcional al subtotal individual de cada persona.
- **REQ-06 (Unwanted Behavior):** SI un ítem no tiene ninguna persona asignada, EL sistema debe excluir su costo del total y mostrar una alerta accesible (`role="alert"`) con el monto pendiente.

### Formato e Interfaz
- **REQ-07 (State-driven):** MIENTRAS la opción "sin decimales" esté activa, EL sistema debe redondear todos los valores calculados al entero más cercano.
- **REQ-08 (State-driven):** MIENTRAS el viewport sea inferior a `768px`, EL sistema debe ocultar la tabla matricial y mostrar los ítems como tarjetas con chips táctiles (toggle de estado visual primario/secundario).
- **REQ-09 (Ubiquitous):** El sistema debe formatear todos los valores monetarios con `Intl.NumberFormat` usando la configuración regional del navegador.

### Liquidación de Deudas
- **REQ-10 (Ubiquitous):** El sistema debe calcular el balance de cada persona: `consumo_total - monto_pagado`.
- **REQ-11 (Event-driven):** CUANDO los balances sean calculados, EL sistema debe ejecutar un algoritmo greedy de minimización de deudas (deudor más grande → acreedor más grande) y renderizar las transferencias resultantes.

### Accesibilidad
- **REQ-12 (Ubiquitous):** El sistema debe usar landmarks semánticos: `<header>`, `<main>`, `<footer>` y secciones con `<section>`/`<article>` donde corresponda.
- **REQ-13 (Ubiquitous):** Todos los `<input>` y `<select>` deben tener un `<label>` vinculado por `for`/`id`. Los botones deben tener nombre accesible explícito.
- **REQ-14 (Ubiquitous):** Los checkboxes y chips de asignación deben ser operables por teclado y comunicar su estado a tecnologías asistivas (estado `checked` nativo o `aria-pressed` para botones toggle).
- **REQ-15 (Ubiquitous):** El contraste de texto debe cumplir WCAG AA (≥4.5:1 para texto normal) en ambos temas (claro y oscuro), delegando al sistema de color de Pico.css v2.

---

# 2. Casos de Uso y Pruebas (Formato Given-When-Then)

### Caso 1: División de ítem con propina proporcional
- **GIVEN:** Ana y Beto. Ítem "Pizza" $10.000 asignado a ambos. Propina 10%.
- **WHEN:** El sistema procesa totales.
- **THEN:** Subtotal Ana $5.000, Beto $5.000. Propina Ana $500, Beto $500. Total individual $5.500 c/u.

### Caso 2: Ítem sin asignar — alerta accesible
- **GIVEN:** Ana y Beto. "Pizza" $10.000 (ambos), "Bebida" $2.000 (sin asignar).
- **WHEN:** El sistema calcula.
- **THEN:** Subtotal Ana $5.000, Beto $5.000. Aparece alerta `role="alert"`: *"Saldo pendiente de $2.000 (Bebida)"*. Total acumulado $10.000.

### Caso 3: Liquidación con múltiples pagadores
- **GIVEN:** Ana consumió $6.000, pagó $10.000 (saldo -$4.000). Beto consumió $4.000, pagó $0 (saldo +$4.000).
- **WHEN:** El sistema ejecuta optimización.
- **THEN:** Una sola instrucción: *"Beto debe transferir $4.000 a Ana"*.

### Caso 4: Importación de estado por URL
- **GIVEN:** URL con hash Base64 de `{"personas":[{"id":1,"nombre":"Carlos"}],"items":[]}`.
- **WHEN:** Alpine.js inicializa.
- **THEN:** Lista muestra solo "Carlos". `localStorage` previo ignorado.

### Caso 5: Modo oscuro automático
- **GIVEN:** El sistema operativo del usuario tiene modo oscuro activo.
- **WHEN:** La página carga.
- **THEN:** La interfaz adopta el tema oscuro sin interacción del usuario, mediante `prefers-color-scheme` gestionado por Pico.css v2 y `<meta name="color-scheme" content="light dark">`.

### Caso 6: Compartir enlace en GitHub Pages
- **GIVEN:** La app está desplegada en `https://<usuario>.github.io/<repo>/`.
- **WHEN:** El usuario copia el enlace y lo abre en otro dispositivo.
- **THEN:** La URL con hash reconstruye el estado idéntico sin necesidad de servidor.

---

# 3. Plan de Implementación (Changelog de Checkpoints)

```markdown
# CHANGELOG & ROADMAP

## [v0.1.0] - Cimiento HTML y Configuración
- [ ] Crear `index.html` con `<!DOCTYPE html>`, `lang="es"`, `charset`, `viewport`, `color-scheme`.
- [ ] Importar Pico.css v2, Open Props (normalize + props) y Alpine.js v3 vía CDN con SRI hashes.
- [ ] Usar variables de Open Props (`--size-*`, `--color-*`, `--shadow-*`, `--ease-*`) para espaciados, sombras y transiciones consistentes.
- [ ] Definir estructura semántica: `<header>`, `<main>`, `<footer>`.
- [ ] Implementar estado global en `x-data="app()"` con estructura mínima (personas, items, config).
- [ ] Panel de configuración: selector de símbolo de moneda y toggle de decimales.

## [v0.2.0] - Gestión de Personas e Ítems
- [ ] Sección de personas: agregar/eliminar con validación de nombre vacío.
- [ ] Sección de ítems: nombre + precio, validación de precio no negativo.
- [ ] Reactividad: al agregar persona, se actualiza la estructura de asignación en cada ítem.
- [ ] Labels vinculados correctamente a todos los inputs.

## [v0.3.0] - Motor de Cálculos y Cargos Adicionales
- [ ] Campos para propina (%) y cargos fijos (delivery, etc.).
- [ ] Funciones reactivas: subtotal individual, distribución proporcional de propina y cargos, total individual.
- [ ] Detección de ítems huérfanos: alerta accesible con `role="alert"` y monto pendiente.

## [v0.4.0] - Interfaz Adaptativa Responsive
- [ ] Vista escritorio (`≥768px`): tabla `<table>` con checkboxes en intersecciones ítem×persona.
- [ ] Vista móvil (`<768px`): tarjetas por ítem con chips tipo `<button>` toggle (aria-pressed).
- [ ] Panel de desglose: subtotales, propina, cargos extra y total por persona.
- [ ] Verificar operabilidad por teclado en ambas vistas.

## [v0.5.0] - Pagos y Liquidación de Deudas
- [ ] Campo "Monto Pagado" por persona.
- [ ] Algoritmo greedy de liquidación: ordenar deudores y acreedores, iterar min(deuda, crédito).
- [ ] Renderizar instrucciones de transferencia en lista semántica `<ol>`.

## [v0.6.0] - Persistencia y Compartición (GitHub Pages Ready)
- [ ] `x-effect` para sincronizar estado completo con `localStorage`.
- [ ] Al iniciar: detectar `location.hash`, decodificar Base64 y restaurar estado (tiene prioridad sobre localStorage).
- [ ] Botón "Copiar Enlace": generar URL `https://<base>/#<base64>` usando `location.origin + location.pathname`, copiar con `navigator.clipboard.writeText()`, mostrar feedback inline (sin alert()).
- [ ] Verificar que la URL generada funcione en GitHub Pages (sin servidor, hash puro).

## [v1.0.0] - Pulido, Accesibilidad y Validación
- [ ] Formateo con `Intl.NumberFormat` adaptado a locale del navegador.
- [ ] Auditoría de accesibilidad: jerarquía de headings, landmarks, contraste, teclado.
- [ ] Pruebas de casos límite: 0 personas, precios vacíos, caracteres especiales en nombres.
- [ ] Verificar dark/light mode en Pico.css v2 sin CSS extra.
- [ ] Despliegue final en GitHub Pages y prueba de URL compartida end-to-end.
```
