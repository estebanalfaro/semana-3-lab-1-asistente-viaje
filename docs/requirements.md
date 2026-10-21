# Requirements Document
## Aplicación Web: Itinerarios y Control de Presupuesto para Viajes

**Versión:** 1.1  
**Fecha:** 2026-10-16  
**Estado:** Borrador  
**Autor:** Analista de Requerimientos Senior

---

## Glosario

| Término | Definición |
|---|---|
| **Viaje** | Entidad principal que agrupa itinerario, presupuesto y gastos de un viaje. |
| **Categoría** | Agrupación temática de gastos (Hospedaje, Tiquetes, Transporte, Actividades, Comida). |
| **Gasto** | Registro de un desembolso económico asociado a una categoría y fecha. |
| **Actividad** | Evento del itinerario manual con fecha, hora opcional y descripción. |
| **ResumenDiario** | Vista calculada del comportamiento de gasto por categoría con control diario en un día específico. |
| **presupuestoGlobal** | Monto total declarado por el usuario para el viaje. |
| **presupuestoRestante** | `presupuestoGlobal − suma de todos los gastos`. Campo calculado, no persistido. |
| **limiteDiario** | `montoAsignado / totalDias`. Solo aplica a categorías con `controlDiario = sí`. |
| **controlDiario** | Indicador booleano. Valor `sí` para Comida y Transporte; `no` para las demás categorías. |

---

## 1. Presupuesto Global y Diario de Comidas

### 1.1 Definición del presupuesto global

**REQ-PG-01**  
WHEN el usuario crea un nuevo viaje  
IF el campo `presupuestoGlobal` tiene un valor numérico mayor o igual a cero  
THEN the system SHALL almacenar el valor como presupuesto total del viaje y utilizarlo como base para el cálculo de `presupuestoRestante`.

**REQ-PG-02**  
WHEN el usuario registra o elimina un gasto  
IF el viaje asociado existe y tiene un `presupuestoGlobal` definido  
THEN the system SHALL recalcular `presupuestoRestante` como `presupuestoGlobal − suma de todos los gastos del viaje` y reflejar el resultado en la interfaz en tiempo real.

**REQ-PG-03**  
WHEN el usuario consulta el resumen del viaje  
IF el viaje tiene categorías con gastos registrados  
THEN the system SHALL mostrar el `presupuestoRestante` actualizado junto al desglose por categoría.

### 1.2 Presupuesto diario por categoría (control diario)

**REQ-PD-01**  
WHEN el sistema calcula el límite diario de una categoría  
IF la categoría tiene `controlDiario = sí` y `totalDias > 0`  
THEN the system SHALL calcular `limiteDiario` como `montoAsignado / totalDias` y presentarlo en el `ResumenDiario` correspondiente.

**REQ-PD-02**  
WHEN el usuario registra un gasto en una fecha determinada para una categoría con `controlDiario = sí`  
IF la suma de gastos de esa categoría en esa fecha supera el `limiteDiario`  
THEN the system SHALL marcar `excedeMeta = sí` y mostrar una alerta visual al usuario.

**REQ-PD-03**  
WHEN el sistema genera el `ResumenDiario` para una fecha dentro del rango del viaje  
IF la categoría es Comida o Transporte  
THEN the system SHALL calcular `saldoDelDia` como `limiteDiario − gastadoDelDia` y mostrarlo con diferenciación visual positiva (saldo disponible) o negativa (exceso).

**REQ-PD-04**  
WHEN el usuario consulta el monto diario disponible de la categoría Comida para una fecha dada  
THEN the system SHALL calcular explícitamente el disponible de ese día utilizando la fórmula:  
`disponibleDia = (Presupuesto Total Comida / Días Totales del Viaje) − Acumulado Gastado en Comida para esa Fecha`  
donde *Presupuesto Total Comida* es el `montoAsignado` de la categoría Comida, *Días Totales del Viaje* es `totalDias`, y *Acumulado Gastado en Comida para esa Fecha* es la suma de todos los gastos de categoría Comida registrados en la fecha consultada, independientemente de si es una fecha pasada, la fecha actual o una fecha futura dentro del rango del viaje.

### 1.3 Recordatorio diario de saldo

**REQ-RD-01**  
WHEN el sistema detecta que la fecha actual corresponde a un día dentro del rango `fechaInicio – fechaFin` de un viaje activo  
IF el usuario tiene la aplicación activa o habilitó notificaciones  
THEN the system SHALL emitir una notificación/recordatorio matutino informando el `presupuestoRestante` disponible para ese día.

---

## 2. Validaciones de Borde

### 2.1 Entidad Viaje

**REQ-VB-01**  
WHEN el usuario intenta guardar un nuevo viaje  
IF el campo `nombre` está vacío o contiene únicamente espacios en blanco  
THEN the system SHALL rechazar el guardado y mostrar el mensaje de error: *"El nombre del viaje no puede estar vacío."*

**REQ-VB-02**  
WHEN el usuario intenta guardar un nuevo viaje  
IF la `fechaFin` es anterior a la `fechaInicio`  
THEN the system SHALL rechazar el guardado y mostrar el mensaje de error: *"La fecha de fin no puede ser anterior a la fecha de inicio."*

**REQ-VB-03**  
WHEN el usuario intenta guardar un nuevo viaje  
IF el campo `presupuestoGlobal` contiene un valor negativo o no numérico  
THEN the system SHALL rechazar el guardado y mostrar el mensaje de error: *"El presupuesto global debe ser un número mayor o igual a cero."*

**REQ-VB-13**  
WHEN el usuario intenta modificar las fechas (`fechaInicio` o `fechaFin`) de un viaje ya creado  
IF existe al menos una actividad cuya `fecha` quedaría fuera del nuevo rango de fechas del viaje  
THEN the system SHALL rechazar la modificación y mostrar el mensaje de error: *"Debe verificar y eliminar las actividades fuera del rango de fecha del viaje."*

### 2.2 Entidad Categoría

**REQ-VB-04**  
WHEN el usuario intenta agregar una categoría a un viaje  
IF el nombre de la categoría ya existe dentro del mismo viaje  
THEN the system SHALL rechazar la operación y mostrar el mensaje de error: *"Ya existe una categoría con ese nombre en este viaje."*

**REQ-VB-05**  
WHEN el usuario intenta guardar una categoría  
IF el campo `nombre` está vacío  
THEN the system SHALL rechazar el guardado y mostrar el mensaje de error: *"El nombre de la categoría no puede estar vacío."*

**REQ-VB-06**  
WHEN el usuario intenta guardar una categoría  
IF el campo `montoAsignado` es negativo o no numérico  
THEN the system SHALL rechazar el guardado y mostrar el mensaje de error: *"El monto asignado debe ser un número mayor o igual a cero."*

### 2.3 Entidad Gasto

**REQ-VB-07**  
WHEN el usuario intenta registrar un gasto  
IF el campo `monto` es cero, negativo, o tiene más de dos decimales  
THEN the system SHALL rechazar el registro y mostrar el mensaje de error: *"El monto debe ser mayor que cero y tener máximo dos decimales."*

**REQ-VB-08**  
WHEN el usuario intenta registrar un gasto  
IF el campo `fecha` está vacío o contiene una fecha fuera del formato `AAAA-MM-DD`  
THEN the system SHALL rechazar el registro y mostrar el mensaje de error: *"La fecha del gasto es obligatoria y debe tener el formato AAAA-MM-DD."*

**REQ-VB-09**  
WHEN el usuario intenta registrar un gasto  
IF el campo `categoria` referencia un nombre que no existe en el viaje actual  
THEN the system SHALL rechazar el registro y mostrar el mensaje de error: *"La categoría seleccionada no existe en este viaje."*

**REQ-VB-14**  
WHEN el usuario intenta eliminar un gasto  
IF el gasto tiene una categoría asociada con al menos un registro de gasto  
THEN the system SHALL mostrar una alerta de confirmación al usuario antes de ejecutar la eliminación, y una vez confirmada, SHALL eliminar el gasto, recalcular el presupuesto gastado de la categoría y actualizar el saldo disponible en la interfaz sin recargar la aplicación.

### 2.4 Entidad Actividad

**REQ-VB-10**  
WHEN el usuario intenta guardar una actividad  
IF el campo `descripcion` está vacío  
THEN the system SHALL rechazar el guardado y mostrar el mensaje de error: *"La descripción de la actividad no puede estar vacía."*

**REQ-VB-11**  
WHEN el usuario intenta guardar una actividad  
IF el campo `hora` tiene un valor con formato diferente a `HH:MM` en 24 horas  
THEN the system SHALL rechazar el guardado y mostrar el mensaje de error: *"La hora debe tener el formato HH:MM en 24 horas (ej. 09:00)."*

**REQ-VB-12**  
WHEN el usuario intenta guardar una actividad  
IF el campo `fecha` está fuera del rango `fechaInicio – fechaFin` del viaje  
THEN the system SHALL rechazar el guardado y mostrar el mensaje de error: *"La fecha de la actividad debe estar dentro del rango del viaje."*

---

## 3. Persistencia de Datos Local

**REQ-PER-01**  
WHEN el usuario crea, edita o elimina un Viaje, Categoría, Gasto o Actividad  
IF la operación supera todas las validaciones de borde  
THEN the system SHALL persistir el cambio en el almacenamiento local del navegador (`localStorage` o `IndexedDB`) antes de confirmar la operación al usuario.

**REQ-PER-02**  
WHEN el usuario recarga o vuelve a abrir la aplicación  
IF existe información previamente guardada en el almacenamiento local  
THEN the system SHALL restaurar automáticamente el estado completo de los viajes, categorías, gastos y actividades sin requerir acción del usuario.

**REQ-PER-03**  
WHEN el sistema persiste un viaje o cualquiera de sus entidades relacionadas  
IF el modelo de datos incluye campos calculados (`presupuestoRestante`, `gastado`, `restante`, `limiteDiario`, `gastadoDelDia`, `saldoDelDia`, `excedeMeta`, `totalDias`)  
THEN the system SHALL omitir dichos campos del almacenamiento y recalcularlos en tiempo de ejecución cada vez que se requieran.

**REQ-PER-04**  
WHEN el usuario solicita eliminar un viaje  
IF el viaje contiene categorías, gastos o actividades asociadas  
THEN the system SHALL eliminar en cascada todos los registros relacionados del almacenamiento local y notificar al usuario que la acción es irreversible antes de ejecutarla.

---

## 4. Restricciones de Diseño (Unchanged Behavior)

Estas restricciones definen comportamientos que el sistema **NO debe modificar** bajo ninguna circunstancia, independientemente de futuras iteraciones o solicitudes de cambio.

**REQ-UB-01**  
WHEN el usuario accede a cualquier funcionalidad de gestión de itinerarios  
IF la solicitud involucra la generación automática de rutas, destinos o actividades  
THEN the system SHALL rechazar la solicitud y no ejecutar ninguna lógica de recomendación o generación automática de itinerarios.  
> *Nota del cliente: La aplicación NO debe generar itinerarios automáticos ni recomendar destinos/actividades; únicamente debe permitir la gestión manual del itinerario.*

**REQ-UB-02**  
WHEN el sistema procesa cualquier entidad del modelo de datos  
IF los campos `presupuestoRestante`, `totalDias`, `gastado`, `restante`, `limiteDiario`, `gastadoDelDia`, `saldoDelDia` y `excedeMeta` son requeridos para una vista o cálculo  
THEN the system SHALL calcularlos en tiempo de ejecución y nunca persistirlos en el almacenamiento local.

**REQ-UB-03**  
WHEN el sistema asigna el valor de `controlDiario` a una categoría  
IF el nombre de la categoría es **Comida** o **Transporte**  
THEN the system SHALL asignar `controlDiario = sí` de forma fija, sin permitir que el usuario lo modifique.

**REQ-UB-04**  
WHEN el sistema asigna el valor de `controlDiario` a una categoría  
IF el nombre de la categoría es **Hospedaje**, **Tiquetes** o **Actividades**  
THEN the system SHALL asignar `controlDiario = no` de forma fija, sin permitir que el usuario lo modifique.

**REQ-UB-05**  
WHEN el sistema valida el nombre de una nueva categoría  
IF el nombre ingresado no corresponde a ninguno de los valores permitidos: Hospedaje, Tiquetes, Transporte, Actividades, Comida  
THEN the system SHALL rechazar la categoría y mostrar al usuario la lista de nombres válidos.

---

## Matriz de Trazabilidad

| ID | Origen | Sección | Prioridad | Versión |
|---|---|---|---|---|
| REQ-PG-01..03 | Regla 2 – Presupuesto y Categorías | §1.1 | Alta | 1.0 |
| REQ-PD-01..03 | Modelo `Categoria.controlDiario` / `ResumenDiario` | §1.2 | Alta | 1.0 |
| REQ-RD-01 | Regla 4 – Recordatorio Diario de Saldo | §1.3 | Media | 1.0 |
| REQ-VB-01..03 | Modelo `Viaje` | §2.1 | Alta | 1.0 |
| REQ-VB-13 | Solicitud de cambio – modificación de fechas con actividades existentes | §2.1 | Alta | 1.1 |
| REQ-VB-04..06 | Modelo `Categoria` | §2.2 | Alta | 1.0 |
| REQ-VB-07..09 | Modelo `Gasto` | §2.3 | Alta | 1.0 |
| REQ-VB-14 | Solicitud de cambio – eliminación de gasto con confirmación y actualización reactiva | §2.3 | Alta | 1.1 |
| REQ-VB-10..12 | Modelo `Actividad` | §2.4 | Media | 1.0 |
| REQ-PER-01..04 | Arquitectura web sin backend | §3 | Alta | 1.0 |
| REQ-UB-01..05 | Nota explícita del cliente / Modelo de datos | §4 | Alta | 1.0 |
