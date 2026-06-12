# Formatos de datos

Tres representaciones del mismo estado. Conocerlas es clave para extender la app sin romper compatibilidad.

## Estado canónico (JSON)

Es lo que viaja a `localStorage` (clave `dividir-cuentas`) y dentro de la URL:

```json
{
  "personas": [{ "id": 1, "nombre": "Ana", "pagado": 30 }],
  "items": [
    {
      "id": 1,
      "nombre": "Pizza",
      "precio": 120,
      "modo": "igual",          // "igual" | "partes" | "pct"
      "partes": { "1": 2 },      // solo en partes/pct: {personaId: valor}
      "asignados": [1, 2],       // solo relevante en modo igual
      "pagadorItemId": 1         // quién pagó este ítem (null = nadie)
    }
  ],
  "propinaPct": 10,
  "cargosExtra": 5,
  "moneda": "$",
  "sinDecimales": false,
  "pagadorId": 2,                // 💳 pagador global (null = nadie)
  "metodoLiquidacion": "minimo"  // "minimo" | "central" | "proporcional"
}
```

!!! note "Migración hacia adelante"
    `_cargarEstado()` aplica defaults a campos ausentes (`modo: 'igual'`, `partes: {}`, `pagadorItemId: null`), así los estados guardados por versiones viejas siguen cargando. **Al agregar un campo nuevo, dale default ahí** y nunca renombres los existentes.

## URL hash

```
https://manfredengler.github.io/dividir_cuenta/#<base64>
```

El JSON canónico se codifica UTF-8 → Base64 (`TextEncoder` + `btoa`). Al cargar, `init()` lo decodifica, restaura el estado y limpia el hash con `history.replaceState`. La URL tiene prioridad sobre `localStorage`. El QR contiene exactamente esta misma URL.

## localStorage

| Clave | Contenido |
|---|---|
| `dividir-cuentas` | Estado canónico JSON (escrito por `x-effect` en cada cambio) |
| `dc-tema` | `"light"` o `"dark"` (ausente = auto). Separado para que el script anti-flash del `<head>` lo lea sin parsear JSON |

## Excel (.xlsx)

Cuatro hojas. La importación usa los **nombres de columna y de hoja como contrato** — si los cambiás, rompés archivos viejos.

**`Participantes`**

| Nombre | Pagó al local | Paga la cuenta |
|---|---|---|
| Ana | 30 | |
| Beto | 0 | SI |

**`Consumos`** — una columna fija por campo + una columna por participante:

| Ítem | Precio | División | Pagado por | Ana | Beto |
|---|---|---|---|---|---|
| Pizza | 120 | igual | Ana | x | x |
| Vino | 60 | partes | | 2 | 1 |
| Postre | 40 | pct | Beto | 25 | 75 |

En modo `igual` la celda lleva cualquier texto no vacío (asignado); en `partes`/`pct` lleva el número.

**`Config`** — pares Opción/Valor: `Propina %`, `Otros cargos`, `Moneda`, `Sin decimales` (SI/NO), `Método liquidación`.

**`Resumen`** — solo de salida (la importación la ignora): consumo, propina, cargos, total, pagó y balance por persona.
