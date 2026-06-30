---
name: flutter-routing-mastery
description: Senior Flutter developer guidelines for routing, navigation, and page lifecycle management. Trigger this skill whenever discussing page navigation, adding intermediate screens, refactoring routes, or recycling existing pages in the frontend.
---

# Senior Flutter Routing & Navigation Strategy

Como desarrollador Senior de Flutter, la navegación y el enrutamiento no son solo "cambiar de pantalla", sino la columna vertebral de la experiencia de usuario y la gestión del estado. Antes de crear, modificar, refactorizar o reciclar cualquier página o flujo de navegación, **DEBES** ejecutar este análisis exhaustivo.

## 1. Análisis Profundo de la Pantalla (Los 4 Pilares)
Antes de tocar el código de navegación, define y documenta explícitamente:
- **Propósito (Purpose)**: ¿Cuál es el objetivo de negocio y de usuario exacto de esta página? ¿Es un estado transitorio (ej. un *loading*, un paso en un formulario) o un destino principal?
- **Estado (State)**: ¿Qué estado necesita para renderizarse? ¿Qué providers de Riverpod consume o altera? ¿Mantiene estado efímero que deba limpiarse al salir?
- **Origen (Where it comes from)**: ¿Qué pantallas pueden navegar *hacia* esta página? ¿Hay múltiples puntos de entrada? ¿Qué parámetros espera recibir obligatoriamente de su origen?
- **Destino (Where it goes)**: ¿Cuáles son los puntos de salida de esta pantalla? ¿Son acciones irreversibles, sub-flujos anidados o pestañas hermanas? ¿Cómo afecta salir de esta página a la pila de navegación (stack)?

## 2. Reutilización y Refactorización (Reciclaje)
- **Reciclar antes de Reconstruir**: Antes de crear una página nueva, busca en el repositorio si existe una pantalla similar que pueda parametrizarse (ej. pasar un `enum` o un booleano `isReadOnly`) para servir a ambos contextos sin duplicar código.
- **Páginas Intermedias (Middlewares visuales)**: Al insertar pantallas intermedias (ej. confirmaciones, selectores), asegúrate de que no rompan el historial de navegación hacia atrás (back-stack) ni el estado del flujo padre.

## 3. Adaptación a la Arquitectura del Proyecto
- **Respeta el Paradigma de GossipGarden**: Este proyecto utiliza una estrategia de enrutamiento personalizada con un `IndexedStack` y *overlays* (gestionado por `navigation_provider.dart` y `NavigationState`), en lugar del clásico `Navigator.push`.
- **Preservación del Estado**: Asegúrate de que tus cambios respeten la preservación local del estado que provee el `IndexedStack`. No destruyas estado a menos que sea el comportamiento esperado.
- **Manejo Manual del "Back"**: Cuando crees nuevos overlays o vistas anidadas, verifica explícitamente cómo interactuará tu nueva capa con el comportamiento de retroceso (`notifier.handleBack()`). ¡Cuidado con dejar al usuario atrapado!

## 4. Mejores Prácticas del Mercado (State-Driven Routing)
- **Desacoplamiento**: Evita el acoplamiento fuerte. La "Pantalla A" no debería conocer los detalles internos de la "Pantalla B" para navegar hacia ella.
- **Paso de Parámetros Lean**: Pasa solo identificadores estrictamente necesarios (ej. `plantId`) en lugar de pasar objetos de datos pesados entre rutas. Deja que la nueva pantalla lea el objeto completo desde Riverpod o la base de datos usando el ID.
- **Edge Cases (Casos límite)**: Piensa siempre: ¿Qué pasa si el usuario presiona el botón físico de "Atrás" en Android? ¿Qué pasa si la data que se está mostrando se elimina en el servidor mientras la pantalla está abierta?
