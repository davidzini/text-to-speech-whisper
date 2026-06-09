# Contexto para agentes

## Qué es este proyecto

Despliegue mínimo (filosofía **KISS**) de [Speaches](https://speaches.ai/) v0.9.0-rc.3 con Docker Compose
para transcribir audio a texto (Whisper / faster-whisper). UI web con micrófono + API compatible
con OpenAI en http://localhost:8000.

No hay código fuente propio: solo `docker-compose.yml` y documentación. Mantenerlo así;
no agregar servicios, scripts ni abstracciones salvo pedido explícito del usuario.

## Decisiones ya tomadas (no re-discutir)

- **Speaches** elegido sobre whisper-asr-webservice / app propia: 1 contenedor, UI con mic incluida.
- **Versión fijada** (`0.9.0-rc.3`), no usar `latest`. Se subió desde 0.8.3 por bug de Gradio
  con idioma del navegador en español (UI en negro); la API no se veía afectada.
- **Perfiles de compose**: `cpu` para esta máquina, `gpu` para otra máquina con NVIDIA decente.
- **Puerto solo en `127.0.0.1`**: el micrófono del navegador exige localhost o HTTPS. No exponer
  a LAN sin agregar TLS (Caddy/Tailscale).
- **Modelos**: `Systran/faster-whisper-small` en CPU (audios en español),
  `deepdml/faster-whisper-large-v3-turbo-ct2` en GPU.

## Hardware de esta máquina

- 12 hilos de CPU. GPU NVIDIA con driver 470 y ~1 GB de VRAM: **inservible para Whisper**,
  no intentar el perfil `gpu` acá. Tampoco hay nvidia-container-toolkit instalado.

## Operación y verificación

```bash
docker compose --profile cpu up -d      # levantar (en la otra máquina: --profile gpu)
curl http://localhost:8000/health        # debe dar 200

# Los modelos hay que descargarlos UNA vez antes de transcribir:
curl -X POST http://localhost:8000/v1/models/Systran/faster-whisper-small

# Prueba end-to-end rápida (wav de voz que viene con ALSA):
curl http://localhost:8000/v1/audio/transcriptions \
  -F file=@/usr/share/sounds/alsa/Front_Center.wav \
  -F model=Systran/faster-whisper-small
# Esperado: {"text":"Front. Center.","logprobs":null,"usage":null}
```

Los modelos persisten en el volumen `hf-hub-cache`.

## Gotchas conocidos

- **UI en negro + error `svelte-i18n`**: bug de Gradio 5.36.x cuando el idioma preferido del
  navegador no es inglés (común con Firefox/Chrome en español). Workaround sin actualizar:
  poner *English (United States)* primero en los idiomas del navegador y recargar. Fix definitivo:
  imagen >= 0.9.0-rc.2 (Gradio >= 5.49).
- Imagen CUDA por defecto requiere driver NVIDIA >= 570; con drivers más viejos usar los tags
  `0.9.0-rc.3-cuda-12.6.3` o `0.9.0-rc.3-cuda-12.4.1` (comentado en el compose).
- v0.9.x sigue en RC. Hubo reportes de pérdida de soporte de algún formato de audio (ogg)
  entre versiones 0.8.x. Probar la transcripción end-to-end después de cualquier cambio de tag.
