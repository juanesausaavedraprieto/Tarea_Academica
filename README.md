# Tarea_Academica

Proyecto basado en n8n para procesar registros entrantes mediante un webhook, normalizar datos y activar una salida cuando el estado del registro es `urgente`.

## Descripción general

Este repositorio contiene un flujo de automatización para recibir un payload HTTP, limpiarlo, validar su contenido y redirigirlo a una acción posterior según las condiciones definidas. La lógica principal está en el workflow exportado en [workflows/flujo.json](workflows/flujo.json).

El flujo implementa estas fases:

1. Recepción del payload mediante un webhook.
2. Normalización del JSON entrante.
3. Validación de un UUID si viene en la solicitud.
4. Eliminación de campos vacíos y duplicados.
5. Evaluación del campo `status`.
6. Envío de una notificación mock cuando el valor es `urgente`.

## Estructura del repositorio

- [docker-compose.yml](docker-compose.yml): define el contenedor de n8n y el volumen persistente.
- [workflows/flujo.json](workflows/flujo.json): export del workflow del proyecto.
- [.gitignore](.gitignore): excluye archivos temporales, secretos y cachés locales.
- [LICENSE](LICENSE): licencia MIT.

## Requisitos

Antes de iniciar, asegúrate de tener instalado:

- Docker Engine
- Docker Compose v2 o `docker-compose`
- Git
- Un navegador web

## Instalación local

### 1) Clonar el repositorio

```bash
git clone <url-del-repositorio>
cd Tarea_Academica
```

### 2) Levantar n8n con Docker

```bash
docker compose up -d
```

Si tu entorno usa la versión clásica:

```bash
docker-compose up -d
```

### 3) Acceder a la interfaz web

Abre en el navegador:

```text
http://localhost:5678
```

La primera vez, n8n pedirá crear una cuenta o iniciar sesión según la configuración del entorno.

### 4) Importar el workflow

Dentro de la interfaz de n8n:

- ir a “Workflows”
- seleccionar “Import from File”
- cargar el archivo [workflows/flujo.json](workflows/flujo.json)

### 5) Activar el workflow

Una vez importado, activa el flujo para que el webhook quede disponible.

## Cómo probar el webhook

El workflow expone un webhook con la ruta:

```text
registro-entrante
```

La URL completa se genera dentro de n8n al activar el nodo webhook. En un entorno local, suele verse como algo similar a:

```text
http://localhost:5678/webhook/registro-entrante
```

### Ejemplo con curl

```bash
curl -X POST http://localhost:5678/webhook/registro-entrante \
  -H "Content-Type: application/json" \
  -d '{
    "uuid": "123e4567-e89b-42d3-a456-426614174000",
    "status": "urgente",
    "nombre": "Ana",
    "email": "ana@example.com"
  }'
```

### Ejemplo con un registro no urgente

```bash
curl -X POST http://localhost:5678/webhook/registro-entrante \
  -H "Content-Type: application/json" \
  -d '{
    "uuid": "123e4567-e89b-42d3-a456-426614174000",
    "status": "normal",
    "nombre": "Luis",
    "email": "luis@example.com"
  }'
```

En este caso, la condición del flujo no debería disparar la salida de notificación.

## Flujo de procesamiento

El workflow realiza la siguiente lógica:

- Recibe el payload vía webhook.
- Toma el JSON desde `body` o desde el propio payload si llega directamente.
- Valida si `uuid` tiene formato UUID correcto.
- Elimina valores nulos, vacíos y duplicados.
- Mantiene el UUID solo si es válido.
- Comprueba si `status` es igual a `urgente`.
- Si coincide, envía una llamada HTTP mock a `https://example.com/api/notificaciones`.

## Datos persistentes

El volumen Docker configurado en [docker-compose.yml](docker-compose.yml) mantiene la información de n8n para que no se pierda la configuración al reiniciar el contenedor.

## Detener el entorno

```bash
docker compose down
```

o

```bash
docker-compose down
```

## Solución de problemas

- Si la interfaz no responde: verifica que Docker esté ejecutándose.
- Si el webhook no llega: asegúrate de que el flow esté activado.
- Si el payload no se procesa: revisa la estructura del JSON enviado y la ruta del webhook.
- Si no aparece la notificación: confirma que el valor de `status` sea exactamente `urgente`.

## Licencia

Este proyecto se distribuye bajo la licencia MIT. Consulta [LICENSE](LICENSE) para más información.
