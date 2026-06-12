# Divisor de Cuentas

Divide cuentas grupales de forma equitativa, sin servidor y sin que tus datos salgan del navegador.

**[Abrir la aplicación →](https://manfredengler.github.io/dividir_cuenta/)**

## Qué hace

- Cada participante paga **lo que consumió**; propina y cargos se reparten en proporción.
- Tres modos de división por ítem: **igual**, **por partes** y **por porcentaje**.
- Registra quién pagó cada ítem (o la cuenta entera) y genera un **plan de transferencias** para saldar las deudas, con tres métodos a elegir.
- Comparte la cuenta por **enlace**, **panel nativo del teléfono** (WhatsApp, Gmail…) o **código QR** — el estado completo viaja codificado en la URL.
- Exporta e importa todo a **Excel** (`.xlsx`).

## Filosofía

| Principio | Cómo se aplica |
|---|---|
| Archivo único | Toda la app es un `index.html`; no hay build step ni servidor |
| Privacidad | `localStorage` + URL hash; el QR y el Excel se generan localmente |
| Accesible | WCAG AA auditado (axe-core: 0 violaciones), tap targets ≥44px, `<dialog>` nativo |
| Rápido | CSS combinado en un request; QR y SheetJS se cargan solo al usarse |

## Mapa de esta documentación

- **[Guía de uso](guia-de-uso.md)** — flujo completo con ejemplos.
- **[Arquitectura](arquitectura.md)** — cómo está construida: estado Alpine, motor de cálculo, capas de UI.
- **[Formatos de datos](formatos-de-datos.md)** — esquemas de `localStorage`, URL hash y Excel.
- **[Extensibilidad](extensibilidad.md)** — cómo agregar monedas, métodos de liquidación, modos de división o nuevas vistas.
