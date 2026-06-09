# Whisper (Speaches) — audio a texto con Docker

Una instancia de [Speaches](https://speaches.ai/) (faster-whisper) con interfaz web para
grabar desde el micrófono o subir archivos de audio y transcribirlos. Un solo contenedor,
API compatible con OpenAI incluida.

## Requisitos

- Docker con Docker Compose v2.
- **Solo para el perfil GPU**: GPU NVIDIA con driver reciente (>= 570 recomendado) y
  [nvidia-container-toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html) instalado.

## Uso

```bash
# Máquina con CPU (esta)
docker compose --profile cpu up -d

# Máquina con GPU NVIDIA
docker compose --profile gpu up -d
```

Abrir **http://localhost:8000** → pestaña *Speech-to-Text*:

1. Descargar el modelo la primera vez (queda cacheado en un volumen).
2. Grabar con el micrófono o subir un archivo.
3. Transcribir.

Para detener: `docker compose --profile cpu down` (o `gpu`).

## Modelos recomendados (español)

| Hardware | Modelo | Notas |
|---|---|---|
| CPU | `Systran/faster-whisper-small` | Rápido, precisión decente |
| CPU (más precisión) | `Systran/faster-whisper-medium` | ~1-2x la duración del audio |
| GPU | `deepdml/faster-whisper-large-v3-turbo-ct2` | La mejor opción calidad/velocidad |

## API (compatible con OpenAI)

```bash
# Descargar el modelo (solo la primera vez)
curl -X POST http://localhost:8000/v1/models/Systran/faster-whisper-small

# Transcribir
curl http://localhost:8000/v1/audio/transcriptions \
  -F file=@audio.mp3 \
  -F model=Systran/faster-whisper-small \
  -F language=es
```

Documentación de la API: http://localhost:8000/docs

## Windows

Funciona igual con Docker Desktop (backend WSL2), mismos comandos desde PowerShell.
Para GPU solo hace falta el driver NVIDIA de Windows actualizado (el toolkit ya viene
integrado en Docker Desktop).

## Solución de problemas

### La página se ve negra / error `svelte-i18n` en la consola

Bug conocido de Gradio cuando el idioma preferido del navegador es español (u otro distinto
de inglés). La API sigue funcionando; solo falla la UI.

**Opción A (recomendada):** recrear el contenedor con la imagen actual del compose
(`0.9.0-rc.3`), que trae Gradio corregido:

```bash
docker compose --profile cpu up -d --force-recreate
```

**Opción B (sin actualizar):** en el navegador, poner *English (United States)* como primer
idioma (Chrome: `chrome://settings/languages`) y recargar la página.

## Notas

- El micrófono en el navegador solo funciona en `localhost` o vía HTTPS; por eso el
  puerto está publicado solo en `127.0.0.1`. Si algún día querés acceder desde otra
  máquina, hace falta un reverse proxy con TLS (p. ej. Caddy) o Tailscale.
- Los modelos descargados persisten en el volumen `hf-hub-cache`.
