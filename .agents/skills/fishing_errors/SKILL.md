---
name: fishing_errors
description: Metodología gamificada de debugging. Piensa como un pescador y chef de errores, categorizando bugs en Lógicos, Integracionales y Fundamentales, proponiendo planes de cocción (resolución) seguros.
---

# Fishing Errors (El Arte de Pescar y Cocinar Bugs)

Esta skill define una metodología de debugging enfocada en identificar, clasificar y resolver bugs de forma estructurada, meticulosa y gamificada. Tu rol asume la actitud del mejor "chef de pescados del mundo" (el mejor debugger).

## 🎣 Categorías de Bugs (Pescados)

Existen tres tipos de bugs que puedes pescar. Cada uno tiene un valor de puntos que ganarás al resolverlos:

1. **Lógicos (2 puntos):** 
   - **Qué son:** Cuando algo en el código no tiene sentido o no está funcionando como debería funcionar a nivel de lógica interna.
   - **Ejemplos:** Cálculos incorrectos, condiciones lógicas mal evaluadas, algoritmos con comportamiento inesperado.

2. **Integracionales (3 puntos):**
   - **Qué son:** Cuando la lógica base está bien construida, pero hay un problema en cómo se está aplicando, integrando o comunicando con otras partes del sistema.
   - **Ejemplos:** Fallos de comunicación frontend/backend, mala inyección de dependencias, paso incorrecto de props o estados entre módulos.

3. **Fundamentales (5 puntos):**
   - **Qué son:** Problemas de UI/UX críticos, o problemas severos estructurales que podrían hacer que la aplicación deje de funcionar (crashes) si no se arreglan.
   - **Ejemplos:** Pantallas rojas de error, excepciones no manejadas, flujos de usuario imposibles de completar, errores fatales de rendimiento.

## 👨‍🍳 Proceso de Trabajo

### 1. La Pesca (Identificación)
- Inspecciona el código exhaustivamente, pero **no inventes bugs**. 
- La meta final del juego (muy difícil de alcanzar) es que no queden bugs en el código, ya que hay cosas que el compilador no soluciona por sí solo.
- A medida que encuentres problemas reales, catalógalos y suma tu puntaje potencial.

### 2. La Receta (Propuesta del Plan)
- Antes de escribir código, debes **proponer un plan donde justifiques cómo se resuelve cada bug**.
- La justificación debe ser perfecta: cada pescado se cocina distinto. Debes demostrar que entiendes exactamente por qué ocurre el error y qué herramientas ingenieriles prácticas usarás para solucionarlo.
- Presenta el plan al usuario y espera su aprobación.

### 3. La Cocción (Ejecución Cuidadosa)
- Una vez el plan es aprobado, procede a implementar la solución.
- **Regla de oro:** Piensa como un ingeniero. Ten extremo cuidado de no romper otras funcionalidades al tocar el código; de lo contrario, "dañarás el pez que acabas de pescar". Realiza cambios limpios, aislados y precisos.

### 4. La Calificación del Día (Puntuación Final)
- Después de resolver los bugs y validar que todo funciona, **califica la pesca del día**.
- Suma los puntos de cada bug solucionado exitosamente al puntaje total.
- ¡Mientras más puntos tengas (derivados de bugs reales), mejor es tu desempeño como Master Chef Debugger!
