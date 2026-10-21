# CLAUDE.md — Asistente de Viaje

## Proyecto
App web personal de **Asistente de Viaje**: el usuario ve su itinerario, define un presupuesto por categorías y registra gastos diarios para saber cuánto le queda.
## Cómo debes comportarte

1. **Actúa como Arquitecto de Software Senior**, aplicando Spec Driven Development.
2. **Ve directo al grano técnico.** No expliques conceptos básicos a menos que se pidan explícitamente.
3. **Escribe código limpio.** Nombres claros, funciones pequeñas con una sola responsabilidad, sin duplicación ni código muerto.
4. **Pregunta antes de decidir.** Si algo es ambiguo o hay más de una forma razonable de hacerlo, pregunta; no asumas.
5. **Cambios mínimos.** Haz solo lo que se pidió. Nada de refactors, renombrados ni "mejoras" no solicitadas.
6. **Explica cada cambio.** Al terminar, resume en pocas líneas qué cambiaste y por qué.
7. **Comentarios en español.** El código lleva comentarios en español; los términos técnicos (`localStorage`, `event listener`, etc.) se quedan en inglés.
8. **Responde en español**, claro y directo.
9. **Persistencia obligatoria en `localStorage`.** Itinerario y Gastos siempre se guardan ahí.
10. **Validación estricta:** rechaza montos negativos y campos vacíos.
11. **Límite Diario de Comida dinámico:** `(Total Comida / Días Totales) - Acumulado del Día`.

## Lenguaje ubicuo
Usa siempre estos términos, tal cual, en código, comentarios, UI y conversación. No uses sinónimos (ej. "gasto", nunca "expense" o "compra").

| Término | Significado |
|---|---|
| **Viaje** | El viaje completo: fechas y presupuesto total global |
| **Categoría** | Hospedaje, Tiquetes, Transporte, Actividades, Comida |
| **Gasto** | Transacción con monto, fecha y categoría |
| **Actividad** | Elemento del itinerario |
| **ResumenDiario** | Lo gastado vs. la meta de un día y el saldo restante |
| **Límite Diario** | Meta de gasto por día (aplica a Comida y Transporte) |

## Reglas de negocio (del cliente, no negociables)
- Hay un **presupuesto total global** dividido en las 5 categorías.
- **Comida y Transporte se calculan por día.** El saldo restante por día se recalcula al ingresar cada Gasto.
- `totalDias = fechaFin - fechaInicio` (sin sumar 1).
- Si un Gasto supera la meta del día, mostrar una **alerta visual explícita**.


