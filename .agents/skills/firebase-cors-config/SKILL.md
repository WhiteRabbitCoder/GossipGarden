---
name: firebase-cors-config
description: "Configura automáticamente los permisos CORS en Firebase Storage usando firebase_admin. Usar cuando el frontend web (Flutter) tenga problemas cargando imágenes de Firebase por políticas CORS."
---

# Configuración de CORS en Firebase Storage

Esta skill te permite solucionar rápidamente el error recurrente donde el frontend de Flutter Web bloquea las imágenes provenientes de Firebase Storage debido a políticas de CORS (`Access to XMLHttpRequest at '...' has been blocked by CORS policy`).

## Contexto
Flutter Web usa CanvasKit, el cual requiere que el servidor de origen de las imágenes tenga reglas CORS explícitas que permitan solicitudes HTTP `GET`. Firebase Storage (Google Cloud Storage) por defecto no permite esto de forma global.

## Flujo de Solución

1. Entra al directorio del backend (`backendGossipGarden`).
2. Crea o ejecuta un script temporal de Python (`set_cors_temp.py`) que use las credenciales existentes de `firebase-admin` para parchear el bucket:

```python
from app.db.firebase import firebase_db
from firebase_admin import storage

def set_cors():
    try:
        bucket = storage.bucket()
        bucket.cors = [
            {
                "origin": ["*"],
                "method": ["GET", "OPTIONS", "HEAD"],
                "responseHeader": ["Content-Type", "Access-Control-Allow-Origin"],
                "maxAgeSeconds": 3600
            }
        ]
        bucket.patch()
        print(f"CORS rules successfully applied to bucket: {bucket.name}")
    except Exception as e:
        print(f"Error applying CORS: {e}")

if __name__ == "__main__":
    set_cors()
```

3. Ejecuta el script usando el entorno virtual del backend:
   ```bash
   .venv/bin/python set_cors_temp.py
   ```

4. Asegúrate de añadir `set_cors_temp.py` a tu `.gitignore` (o borrar el script una vez ejecutado) para no ensuciar el repositorio.

5. Pídele al usuario que refresque la ventana del navegador (F5) o haga un Hot Restart para que Chrome limpie el caché del bloqueo de CORS.
