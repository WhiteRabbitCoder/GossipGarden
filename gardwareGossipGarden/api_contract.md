# API Contract - Gossip Garden Hardware Module

Este documento describe la interfaz de comunicación del módulo de hardware (ESP32) con el backend a través de MQTT. Al ser un dispositivo IoT, la comunicación se realiza mediante publicación de mensajes (asincrínica) y no mediante peticiones HTTP tradicionales.

## 1. Publicación de Datos de Sensores (Sensor Data Publish)

El ESP32 publica periódicamente las lecturas de los sensores en el broker MQTT para que el backend las procese.

*   **Protocolo:** MQTT
*   **Operación:** PUB (Publish)
*   **Topic (Endpoint):** `plantas/{DEVICE_ID}/sensores`
    *   *Nota:* `{DEVICE_ID}` es el identificador único del hardware, por ejemplo, `esp32_01`.

### Payload Enviado (Envíado por ESP32 al Backend)

El payload es un documento JSON con los datos actuales leídos de los sensores.

```json
{
  "plant_id": "e1db0480-8b23-4302-a752-405a45d311b5",
  "mac_address": "24:6f:28:aa:bb:cc",
  "temperature_c": 24.5,
  "humidity_pct": 45,
  "soil_moisture_pct": 60,
  "light_lux": 350.5
}
```

#### Descripción de los campos

| Campo | Tipo | Descripción | Rango Esperado |
| :--- | :--- | :--- | :--- |
| `plant_id` | String | Identificador único (UUID) de la planta a la que está asignado el dispositivo. | UUID |
| `mac_address` | String | Dirección de hardware (MAC) del módulo WiFi del ESP32. Sirve para identificar el dispositivo físicamente. | `xx:xx:xx:xx:xx:xx` |
| `temperature_c` | Float | Temperatura del aire en grados Celsius (Sensor DHT22). | `-40.0` a `80.0` |
| `humidity_pct` | Integer | Porcentaje de humedad relativa del aire (Sensor DHT22). | `0` a `100` |
| `soil_moisture_pct`| Integer | Porcentaje de humedad del suelo estimado a partir del valor analógico (Sensor SEN0193). | `0` a `100` |
| `light_lux` | Float | Intensidad de luz ambiental en Luxes (Sensor GY-30/BH1750). | `0.0` a `65535.0` |

### Datos Recibidos

*Actualmente, el módulo de hardware de Gossip Garden funciona únicamente como **Productor** (Publisher) en este topic y no suscribe a comandos externos.*

## 2. Configuración de Wi-Fi (HTTP Local)

Si el dispositivo no logra conectarse a una red conocida, iniciará en modo **Punto de Acceso (AP)** con un servidor HTTP embebido (SSID por defecto: `GossipGarden_Setup`, sin contraseña). Se utiliza para la configuración inicial de red desde una aplicación móvil o navegador.

Todos los endpoints HTTP locales devuelven un objeto JSON. La IP por defecto del Access Point del ESP32 suele ser `192.168.4.1`, por lo que la base URL sería `http://192.168.4.1`.

### 2.0. Información del Sistema (Nuevo)

*   **Endpoint:** `/system/info`
*   **Método:** `GET`
*   **Descripción:** Retorna información del hardware local, principalmente la MAC Address para vincular el sensor en la app. Este endpoint no requiere estar conectado a internet, se responde desde el modo AP.

**Respuesta Exitosa (200 OK):**
```json
{
  "mac_address": "24:6f:28:aa:bb:cc",
  "firmware_version": "1.0.0"
}
```

### 2.1. Escaneo de Redes Wi-Fi

*   **Endpoint:** `/wifi/networks`
*   **Método:** `GET`
*   **Descripción:** Retorna una lista con las redes Wi-Fi visibles para el dispositivo.

**Respuesta Exitosa (200 OK):**
```json
[
  {
    "ssid": "MiRedCasa",
    "bssid": "aa:bb:cc:dd:ee:ff",
    "channel": 6,
    "rssi": -65,
    "security": "WPA2-PSK",
    "hidden": false
  }
]
```

### 2.2. Estado de la Conexión Wi-Fi

*   **Endpoint:** `/wifi/status`
*   **Método:** `GET`
*   **Descripción:** Informa si el dispositivo está actualmente conectado como estación (STA).

**Respuesta Exitosa (200 OK):**
```json
{
  "connected": true,
  "ip": "192.168.1.130",
  "status_code": 1010
}
```

### 2.3. Conexión a una Nueva Red

*   **Endpoint:** `/wifi/connect`
*   **Método:** `POST`
*   **Descripción:** Intenta conectarse a una red Wi-Fi y, si resulta exitosa, la guarda de forma permanente y apaga el Modo AP.

**Payload Requerido (JSON):**
```json
{
  "ssid": "MiRedCasa",
  "password": "MiContraseñaSecreta"
}
```

**Respuesta Exitosa (200 OK):**
```json
{
  "success": true,
  "ip": "192.168.1.130"
}
```
*   **Nota:** Al tener éxito, el Access Point se desactivará un segundo después y el servidor dejará de contestar las siguientes peticiones.

**Respuesta Fallida (Conexión Denegada / Timeout - 200 OK):**
```json
{
  "success": false,
  "status_code": 202
}
```

**Respuesta Fallida (Formato Incorrecto - 400 Bad Request):**
```json
{
  "error": "Error de parseo JSON"
}
```

### 2.4. Reseteo de Credenciales y Desconexión

*   **Endpoint:** `/wifi/reset`
*   **Método:** `POST`
*   **Descripción:** Elimina el archivo `wifi_config.json` que guarda las credenciales y fuerza la desconexión del modo estación.

**Respuesta Exitosa (200 OK):**
```json
{
  "status": "reset"
}
```
