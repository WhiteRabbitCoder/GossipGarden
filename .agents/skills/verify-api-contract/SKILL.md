---
name: verify-api-contract
description: Trigger this skill before modifying any frontend data logic, screen state, or API integration. It verifies that the frontend strictly matches the backend data structures, types, and endpoints as defined in the API_CONTRACT.md to prevent data inconsistencies.
---

# Verificación de Contrato API e Integración de Datos (Frontend/Backend)

Antes de realizar cualquier modificación lógica en el frontend que involucre consumo, envío o manipulación de datos, **DEBES** ejecutar este flujo de validación de manera obligatoria. El objetivo es evitar inconsistencias, errores de tipado o problemas de formato entre el cliente y el servidor.

## Pasos Obligatorios:

### 1. Entender el Contexto de la Pantalla (Frontend)
- Analiza el código de la pantalla o widget a modificar.
- Identifica qué datos exactos necesita consumir o mostrar.
- Identifica qué `provider`, `datasource` o repositorio maneja esta información.
- Identifica los endpoints exactos que se están consumiendo.

### 2. Verificar el Contrato (API_CONTRACT.md)
- Lee el archivo `API_CONTRACT.md` (disponible en la raíz de `backendGossipGarden` o `frontendGossipGarden`).
- Localiza el endpoint específico involucrado.
- Toma nota estricta de:
  - Estructura del JSON (Request y Response).
  - Tipos de datos (ej. `String`, `int`, `double`, `bool`, etc.).
  - Nulabilidad (campos opcionales vs. requeridos).
  - Formatos especiales (ej. fechas en ISO 8601, URLs, enums).

### 3. Entender la Fuente de la Verdad (Backend)
- Si el `API_CONTRACT.md` no es suficiente o hay dudas, revisa el origen de los datos en el backend:
  - Archivos Pydantic en `backendGossipGarden/app/schemas/`.
  - Esquema de base de datos en `backendGossipGarden/migrations/schema.sql` (para entender qué tablas alimentan este endpoint).
- Comprende la lógica de negocio subyacente (ej. campos calculados, campos generados por joins).

### 4. Ejecutar la Integración / Modificación
- Compara tu entendimiento del backend con el modelo (`model`) o DTO actual en el frontend.
- Antes de agregar lógica en la UI, actualiza o crea los modelos en Dart para que sean un reflejo **1:1** de lo que dicta el contrato.
- Presta especial atención al parseo de JSON (uso de `.fromJson` o métodos similares).
- Una vez asegurada la consistencia estricta de tipos y formatos, procede a escribir o modificar la lógica en la pantalla solicitada.
