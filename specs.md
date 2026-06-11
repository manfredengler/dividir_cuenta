A continuación, se presentan la definición del proyecto, las especificaciones técnicas estructuradas en formato **EARS (Easy Approach to Requirements Syntax)**, los casos de uso/prueba en formato **Given-When-Then**, y el plan de implementación estructurado como un **Changelog de Checkpoints**.

---

# Definición y Resumen del Proyecto

**Divisor de Cuentas Standalone**  
Un archivo único HTML ejecutable en el navegador localmente (offline) que facilita la división detallada de consumos grupales. Utiliza **Alpine.js** para la lógica reactiva y **Pico.css** para un diseño adaptativo limpio y optimizado para móviles.

### Características Principales:
*   **División por ítems:** Los productos se dividen equitativamente solo entre quienes los consumieron.
*   **Cargos mixtos:** Soporta propinas (%) y cargos fijos (delivery) distribuidos de forma proporcional.
*   **Diseño adaptativo:** Tabla interactiva en escritorio; tarjetas táctiles (chips) en móviles.
*   **Sin servidor (Serverless):** Persistencia en `localStorage` y compartición mediante codificación de datos en la URL (Base64).
*   **Múltiples pagadores:** Algoritmo que simplifica deudas determinando quién debe transferir cuánto a quién.

---

# 1. Especificaciones de Requisitos (Formato EARS)

El formato EARS utiliza patrones específicos según el tipo de requisito:
*   *Ubiquitous (Siempre activo):* "El sistema debe..."
*   *Event-driven (Disparado por evento):* "CUANDO [evento] EL sistema debe..."
*   *State-driven (Dependiente de un estado):* "MIENTRAS [estado] EL sistema debe..."
*   *Unwanted Behavior (Manejo de errores/excepciones):* "SI [error/caso límite] EL sistema debe..."
*   *Optional (Característica opcional):* "DONDE [opción habilitada] EL sistema debe..."

### Gestión de Datos y Estado
*   **REQ-01 (Ubiquitous):** El sistema debe almacenar el estado actual de las personas, ítems, cargos y configuraciones en el `localStorage` del navegador ante cualquier cambio.
*   **REQ-02 (Event-driven):** CUANDO el usuario haga clic en "Copiar Enlace", EL sistema debe codificar el JSON del estado actual en Base64, generar una URL con dicho hash y copiarla al portapapeles.
*   **REQ-03 (Event-driven):** CUANDO la aplicación se cargue y detecte un hash en la URL, EL sistema debe decodificar el hash Base64 e instanciar el estado de la aplicación con esos datos, ignorando el `localStorage`.

### Cálculos y Distribución
*   **REQ-04 (Ubiquitous):** El sistema debe calcular el consumo individual de cada persona dividiendo el precio de cada ítem entre el número de comensales seleccionados en dicho ítem.
*   **REQ-05 (Ubiquitous):** El sistema debe distribuir los cargos fijos y la propina de forma directamente proporcional al consumo individual acumulado de cada persona.
*   **REQ-06 (Unwanted Behavior):** SI un ítem no tiene ninguna persona asignada, EL sistema debe excluir su costo de los subtotales individuales y mostrar una alerta visual indicando el monto no asignado.

### Formato e Interfaz
*   **REQ-07 (State-driven):** MIENTRAS la opción de decimales esté configurada en "sin decimales", EL sistema debe aproximar todos los valores calculados al entero más cercano utilizando redondeo matemático estándar.
*   **REQ-08 (State-driven):** MIENTRAS la pantalla del dispositivo sea inferior a `768px` (móvil), EL sistema debe ocultar la tabla matricial y mostrar el listado de ítems en formato de tarjetas con selectores de tipo chip.

### Liquidación de Deudas
*   **REQ-09 (Ubiquitous):** El sistema debe calcular el balance de cada persona restando lo que consumió de lo que declaró haber pagado.
*   **REQ-10 (Event-driven):** CUANDO los balances individuales sean calculados, EL sistema debe ejecutar el algoritmo de minimización de deudas para generar el listado de transferencias optimizado.

---

# 2. Casos de Uso y Pruebas (Formato Given-When-Then)

### Caso de Uso 1: División de un ítem con propina proporcional
*   **GIVEN (Dado que):** 
    *   Hay dos personas: "Ana" y "Beto".
    *   Hay un ítem "Pizza" de $10.000 asignado a ambos.
    *   La propina está configurada al 10%.
    *   No hay otros cargos fijos ni montos pagados previamente.
*   **WHEN (Cuando):**
    *   El sistema procesa los totales.
*   **THEN (Entonces):**
    *   El subtotal para Ana debe ser $5.000 y para Beto $5.000.
    *   La propina asignada a Ana debe ser $500 y a Beto $500.
    *   El total individual a pagar para cada uno debe ser $5.500.

### Caso de Uso 2: Ítem sin asignar (Alerta de saldo pendiente)
*   **GIVEN (Dado que):**
    *   Hay dos personas: "Ana" y "Beto".
    *   Hay dos ítems: "Pizza" de $10.000 (asignado a Ana y Beto) y "Bebida" de $2.000 (sin asignar a nadie).
*   **WHEN (Cuando):**
    *   El sistema realiza el cálculo.
*   **THEN (Entonces):**
    *   El subtotal de Ana debe ser $5.000 y el de Beto $5.000.
    *   El sistema debe mostrar un mensaje de alerta: *"Hay un saldo pendiente de asignación de $2.000 (Bebida)"*.
    *   El total cobrado acumulado debe reflejar $10.000, indicando una discrepancia de $2.000 respecto al total real de la cuenta ($12.000).

### Caso de Uso 3: Liquidación de deudas con múltiples pagadores
*   **GIVEN (Dado que):**
    *   Ana consumió $6.000 y pagó $10.000 al local (Saldo: -$4.000, le deben).
    *   Beto consumió $4.000 y pagó $0 al local (Saldo: +$4.000, debe).
*   **WHEN (Cuando):**
    *   El sistema ejecuta la optimización de pagos.
*   **THEN (Entonces):**
    *   La sección de liquidación debe mostrar exactamente una instrucción: *"Beto debe transferir $4.000 a Ana"*.

### Caso de Uso 4: Importación de estado mediante URL
*   **GIVEN (Dado que):**
    *   El usuario abre la aplicación con el hash de URL: `#eyJwZXJzb25hcyI6W3siaWQiOjEsIm5vbWJyZSI6IkNhcmxvcyJ9XSwiaXRlbXMiOltdfQ==` (JSON codificado: `{"personas":[{"id":1,"nombre":"Carlos"}],"items":[]}`).
*   **WHEN (Cuando):**
    *   La aplicación inicializa el ciclo de vida de Alpine.js.
*   **THEN (Entonces):**
    *   La lista de personas debe mostrar únicamente a "Carlos".
    *   Cualquier dato previo guardado en el `localStorage` de ese dispositivo debe ser ignorado durante esta carga inicial.

---

# 3. Plan de Implementación (Changelog de Checkpoints)

```markdown
# CHANGELOG & ROADMAP DE IMPLEMENTACIÓN

## [v0.1.0] - Cimiento Standalone y Configuración Inicial
- [ ] Crear estructura básica del archivo `index.html`.
- [ ] Importar dependencias vía CDN de forma localizable (Pico.css v2 y Alpine.js v3).
- [ ] Definir estructura del estado global en Alpine.js (`x-data="app()"`).
- [ ] Implementar panel de configuraciones globales:
  - Selector de símbolo de moneda ($, €, etc.).
  - Toggle para activar/desactivar decimales (0 vs 2 decimales).
- [ ] Verificar consistencia de renderizado base de Pico.css en pantallas móviles y escritorio.

## [v0.2.0] - Gestión de Entidades (Personas e Ítems)
- [ ] Crear sección para agregar/eliminar participantes (Personas) con validación de nombres vacíos.
- [ ] Crear sección para agregar/eliminar consumos (Ítems) con campos para Nombre y Precio.
- [ ] Implementar reactividad: al añadir una persona, se actualizan las estructuras de relación en los ítems.
- [ ] Crear las directivas de Alpine.js para sincronizar los checkboxes de asignación.

## [v0.3.0] - Motor de Cálculos y Costos Adicionales
- [ ] Implementar campos para costos adicionales:
  - Propina en porcentaje (input numérico).
  - Otros cargos fijos (input numérico para delivery, servicio, etc.).
- [ ] Escribir funciones reactivas de cálculo:
  - Subtotal individual por consumos directos.
  - Distribución proporcional de la propina sobre el subtotal de cada persona.
  - Distribución proporcional de cargos fijos sobre el subtotal de cada persona.
  - Suma total individual.
- [ ] Desarrollar lógica de detección de ítems huérfanos (sin asignar) y su respectivo panel de advertencia.

## [v0.4.0] - Interfaz Adaptativa (Responsive Layout)
- [ ] Implementar vista de escritorio usando una tabla de doble entrada `<table class="striped">`:
  - Filas: Ítems.
  - Columnas: Personas (con checkboxes en las intersecciones).
- [ ] Implementar vista móvil usando clases de rejilla de Pico.css (`grid`):
  - Tarjetas individuales para cada ítem.
  - Botones de selección táctil ("chips") que cambien de color secundario a primario al estar activos.
- [ ] Agregar el panel inferior con el desglose detallado de subtotales, propinas, cargos extras y totales individuales por persona.

## [v0.5.0] - Registro de Pagos y Liquidación de Deudas
- [ ] Añadir campo "Monto Pagado" al perfil de cada persona.
- [ ] Desarrollar el algoritmo de liquidación de deudas:
  - Identificar saldos individuales (`Consumo Total - Pagado`).
  - Dividir la lista en dos grupos: Deudores (Saldo > 0) y Acreedores (Saldo < 0).
  - Resolver deudas iterando desde el deudor más grande hacia el acreedor más grande.
- [ ] Renderizar las instrucciones paso a paso de transferencia (ej: "X le paga $Y a Z").

## [v0.6.0] - Persistencia, Serialización e Intercambio
- [ ] Configurar observador `x-effect` de Alpine.js para sincronizar el estado completo con `localStorage` ante cualquier cambio.
- [ ] Implementar lógica de lectura de hash URL al arrancar el componente para restaurar estados compartidos.
- [ ] Crear botón "Compartir Cuenta":
  - Generar JSON estructurado.
  - Convertir JSON a string comprimido/codificado en Base64.
  - Insertar en el portapapeles el enlace generado de forma silenciosa e informar al usuario con un tooltip o banner de Pico.css.

## [v1.0.0] - Pulido y Validación
- [ ] Aplicar formato de moneda localizado utilizando la API `Intl.NumberFormat` integrada de forma reactiva.
- [ ] Realizar pruebas manuales de casos límite (0 personas, precios vacíos, caracteres especiales).
- [ ] Optimizar espaciados visuales con Pico.css para mejorar legibilidad en modo oscuro y claro nativos de Pico.css.
- [ ] Entregar el archivo único HTML consolidado, sin dependencias de compilación adicionales.
```
