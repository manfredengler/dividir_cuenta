# Extensibilidad

Recetas para las extensiones más probables. Regla general: **el estado canónico es el contrato** — agregá campos con default en `_cargarEstado()`, no renombres nada, y el resto (localStorage, URL, QR) se mantiene compatible solo.

## Agregar una moneda

Dos `<select>` duplicados (navbar escritorio y drawer móvil) comparten `x-model="moneda"`. Agregá la misma `<option>` en ambos:

```html
<option value="¥">¥ Yen</option>
```

Nada más: `fmt()` antepone el símbolo al número formateado por `Intl.NumberFormat`.

## Agregar un método de liquidación

1. **Algoritmo** — en `_liquidar(saldos)` agregá una rama. Recibís `saldos` (`{id, nombre, total, pagado, saldo}`; saldo > 0 = debe) y devolvés `[{de, para, monto, detalle}]`.
2. **Chip** — agregá el botón en la sección "Liquidación de deudas" siguiendo el patrón `:aria-pressed` + `@click="metodoLiquidacion = 'tuMetodo'"`.
3. **Descripción** — agregá la entrada en el diccionario de `descMetodo()`.
4. **Validación de import** — sumá el nombre a la lista blanca en `importarExcel()` (`['minimo','central','proporcional']`).

## Agregar un modo de división

Más invasivo; los puntos a tocar:

1. `get resumen()` — la rama de cálculo del nuevo modo (cómo convierte `item.partes` en montos).
2. Los dos `<select>` de modo (tabla escritorio y editor móvil) + `alCambiarModo()` si necesita siembra inicial.
3. `resumenItem()` — cómo se describe en la tarjeta compacta móvil.
4. Export/import Excel: decidir qué va en la celda por persona y parsearlo de vuelta.
5. Lista blanca de modos en `importarExcel()` (`['igual','partes','pct']`).

## Agregar una vista o sección

Las secciones de `<main>` se numeran solas (CSS counter en `section > h2::before`). Una sección nueva:

```html
<section aria-labelledby="h-mia">
  <h2 id="h-mia">Mi sección</h2>
  ...
</section>
```

Para modales usá el patrón existente: `<dialog>` + `showModal()` envuelto en `_conTransicion()` para la animación, y un botón de cierre `rel="prev"` (ya estilizado a 44×44px).

## Agregar una dependencia

Seguí el patrón de carga perezosa (ver `_cargarXLSX()` o `mostrarQR()`): inyectar el `<script>` del CDN la primera vez que se usa, con `await` y aviso por `this._avisar()` si falla. **No agregues `<script src>` bloqueante al `<head>`** — el presupuesto de performance ya está al límite con el CSS.

## Checklist antes de publicar un cambio

- [ ] ¿Estados guardados viejos siguen cargando? (default en `_cargarEstado`)
- [ ] ¿Funciona en la tabla escritorio **y** en el editor móvil?
- [ ] ¿Controles nuevos con `aria-label`/`label` y ≥44px en táctil?
- [ ] ¿Se persiste? (si es estado, agregalo a `persistir()`, `_urlEstado()` y al Excel)
- [ ] Auditar: axe-core sin violaciones nuevas, Lighthouse móvil sin regresión
- [ ] Documentar en el changelog del `README.md` (y aquí, si cambia un contrato)

## Cómo se construye esta documentación

```bash
# fuente en docs-src/, salida en docs/ (se comete al repo)
uvx --with mkdocs-material mkdocs build
```

GitHub Pages (modo legacy, rama `main`, carpeta raíz) sirve `docs/` como subruta del mismo sitio: `…/dividir_cuenta/docs/`. No hay workflow aparte: **construir y cometer `docs/`** es el deploy.
