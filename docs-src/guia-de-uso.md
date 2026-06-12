# Guía de uso

## 1. Participantes

Agregá a cada persona por nombre. Cada fila tiene:

- **Pagó** — lo que esa persona ya puso en el local (efectivo, su tarjeta, etc.).
- **💳 (pagador)** — marca a quien "paga la cuenta": cubre automáticamente todo lo que nadie declaró y recibe las transferencias en el método "todo a una persona".
- **×** — eliminarla (se desvincula de todos los ítems).

## 2. Consumos

Cargá cada ítem con nombre y precio. Por defecto queda asignado a todos los participantes en modo **÷ igual**.

### Modos de división

| Modo | Cómo reparte | Ejemplo |
|---|---|---|
| **÷ igual** | Equitativo entre los asignados (checkboxes/chips) | Pizza $120 entre 3 → $40 c/u |
| **partes** | Fracciones ponderadas | Vino: Ana 2, Beto 1 → Ana paga ⅔ |
| **%** | Porcentaje exacto por persona | Postre: Ana 25%, Beto 75% |

!!! warning "Saldo sin asignar"
    Si un ítem queda sin nadie asignado (o el % suma menos de 100), su monto **se excluye del total** y aparece una alerta con el pendiente. Nada se reparte en silencio.

### Pagado por

Cada ítem puede asociarse a quien lo pagó. Ese monto cuenta como pago efectivo de la persona y entra al cálculo de saldos — útil cuando en la mesa cada uno pagó cosas distintas.

### En el teléfono

Los ítems se ven como tarjetas compactas con un resumen ("÷ igual entre Ana, Beto · pagó Ana"). El botón **Editar** abre un editor a pantalla completa con todos los controles.

## 3. Cargos adicionales

Propina (%) y otros cargos fijos (envío, cubierto) se reparten **en proporción al consumo** de cada persona: quien consumió el doble, aporta el doble de propina.

## 4. Resumen y liquidación

El resumen muestra por persona: consumo, propina, cargos, total a pagar, lo pagado y el balance. Cuando hay saldos pendientes aparece la **liquidación de deudas** con tres métodos:

| Método | Qué hace |
|---|---|
| **Mínimas transferencias** | Greedy: el mayor deudor paga al mayor acreedor; la menor cantidad de pagos posible |
| **Todo a una persona** | Todos pagan al 💳 pagador (o mayor acreedor), que devuelve a quienes adelantaron de más |
| **Proporcional** | Cada deudor reparte su pago entre los acreedores según lo que adelantó cada uno |

Cada transferencia explica su cálculo ("Deuda total de Beto: $76 · Ana adelantó $76").

## 5. Compartir y respaldar

- **Compartir** — en el teléfono abre el panel del sistema (WhatsApp, Gmail…); en escritorio copia el enlace. La URL contiene el estado completo: quien la abre ve la cuenta cargada.
- **Código QR** — para pasar la cuenta a alguien que está al lado: escanea y listo.
- **Excel** — descarga un `.xlsx` con participantes, consumos, configuración y resumen; el mismo archivo se puede reimportar para restaurar la cuenta.

!!! tip "Persistencia automática"
    Todo se guarda en `localStorage` con cada cambio. Cerrá la pestaña y volvé: la cuenta sigue ahí.
