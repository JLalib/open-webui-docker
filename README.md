# 🤖 Open WebUI Docker

[![GitHub](https://img.shields.io/badge/GitHub-Repo-blue?logo=github)](https://github.com/JLalib/open-webui-docker)
[![Docker](https://img.shields.io/badge/Docker-ghcr.io%2Fopen--webui%2Fopen--webui-blue?logo=docker)](https://ghcr.io/open-webui/open-webui)
[![License](https://img.shields.io/badge/License-Open%20WebUI%20License-orange)](https://github.com/open-webui/open-webui/blob/main/LICENSE)

## 📋 Descripción general

**Open WebUI** es una interfaz web autohospedada para conversar con modelos de lenguaje (LLM), con una experiencia similar a ChatGPT. Permite conectar modelos locales mediante **Ollama** o usar APIs compatibles con OpenAI (como la de OpenAI, Groq, Together, etc.). Todo se ejecuta en tu servidor: tus conversaciones, usuarios y configuración nunca salen de tu infraestructura.

Este repositorio documenta la instalación y configuración de Open WebUI mediante Docker y Docker Compose, siguiendo la guía oficial y las mejores prácticas para entornos *self-hosted*.

## ✨ Características principales

- 🔗 **Conexión con Ollama**: Usa modelos locales en el mismo servidor o en otro equipo de tu red mediante `OLLAMA_BASE_URL`.
- ☁️ **API compatible con OpenAI**: Configura tu clave con `OPENAI_API_KEY` y usa modelos en la nube desde la misma interfaz.
- ⚡ **Aceleración por GPU**: La imagen `:cuda` aprovecha GPUs Nvidia con `--gpus all`.
- 📴 **Modo sin descargas**: Con `HF_HUB_OFFLINE=1` se desactivan las descargas de Hugging Face, ideal para entornos sin Internet.
- 💾 **Datos en un volumen**: Usuarios, chats y configuración se guardan en el volumen `open-webui`, portable y fácil de respaldar.
- 🔄 **Actualizaciones sencillas**: Descargas la imagen nueva y vuelves a ejecutar el mismo comando; los datos persisten en el volumen.

## 📋 Requisitos del sistema

- ✅ Docker instalado en tu servidor o equipo
- ✅ Puerto **3000** libre en el host (la interfaz escucha en el 8080 dentro del contenedor)
- ✅ Para modelos locales: **Ollama** en el mismo servidor o en otro, y suficiente memoria RAM/VRAM para el modelo elegido
- ✅ Para aceleración por GPU: GPU **Nvidia** con **Nvidia Container Toolkit** instalado

## 🐳 Instalación

### Opción A: Docker CLI (rápida)

```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data --name open-webui --restart always \
  ghcr.io/open-webui/open-webui:main
```

### Opción B: Con Ollama en otro servidor

```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway \
  -e OLLAMA_BASE_URL=http://IP_DE_TU_OLLAMA:11434 \
  -v open-webui:/app/backend/data --name open-webui --restart always \
  ghcr.io/open-webui/open-webui:main
```

### Opción C: Docker Compose (oficial, Open WebUI + Ollama juntos)

```yaml
services:
  ollama:
    volumes:
      - ollama:/root/.ollama
    container_name: ollama
    pull_policy: always
    tty: true
    restart: unless-stopped
    image: ollama/ollama:${OLLAMA_DOCKER_TAG-latest}

  open-webui:
    image: ghcr.io/open-webui/open-webui:${WEBUI_DOCKER_TAG-main}
    container_name: open-webui
    volumes:
      - open-webui:/app/backend/data
    depends_on:
      - ollama
    ports:
      - ${OPEN_WEBUI_PORT-3000}:8080
    environment:
      - 'OLLAMA_BASE_URL=http://ollama:11434'
      - 'WEBUI_SECRET_KEY='
    extra_hosts:
      - host.docker.internal:host-gateway
    restart: unless-stopped

volumes:
  ollama: {}
  open-webui: {}
```

> **Nota**: El compose oficial usa volúmenes Docker gestionados. Si prefieres carpetas en el host, cámbialos por rutas como `./open-webui-data:/app/backend/data`.

Levanta la pila:

```bash
docker compose up -d
```

Descarga un modelo dentro del contenedor de Ollama:

```bash
docker exec -it ollama ollama pull llama3.2
```

## ⚙️ Configuración

1. **Variables de entorno principales**  
   - `OLLAMA_BASE_URL`: URL del servidor Ollama (ej. `http://ollama:11434` o `http://host.docker.internal:11434`).  
   - `OPENAI_API_KEY`: Clave API para proveedores compatibles con OpenAI.  
   - `WEBUI_SECRET_KEY`: Clave secreta para firmar cookies/JWT (genera una aleatoria larga en producción).  
   - `HF_HUB_OFFLINE=1`: Desactiva descargas de Hugging Face (entornos air-gapped).  
   - `OPEN_WEBUI_PORT`: Puerto expuesto en el host (por defecto 3000).

2. **Etiquetas de imagen**  
   | Etiqueta | Qué incluye |
   |----------|-------------|
   | `:main`  | Imagen estándar con dependencias de IA locales |
   | `:slim`  | Más ligera; usa servicios externos para embeddings, extracción de documentos y voz |
   | `:cuda`  | Con aceleración CUDA para GPU Nvidia (requiere `--gpus all`) |
   | `:ollama`| Incluye Ollama dentro del mismo contenedor |
   | `:dev`   | Última versión de desarrollo |

3. **Volúmenes**  
   - `open-webui` → `/app/backend/data` (usuarios, chats, settings, documentos subidos).  
   - `ollama` → `/root/.ollama` (modelos descargados por Ollama).

4. **Red**  
   - `host.docker.internal:host-gateway` permite al contenedor alcanzar servicios en el host (útil si Ollama corre fuera de Docker).

## 🚀 Primeros pasos

1. Ejecuta la instalación (Opción A, B o C).
2. Abre el navegador en `http://localhost:3000` (o `http://IP_DE_TU_SERVIDOR:3000`).
3. Crea la **cuenta de administrador** en la pantalla de registro (la primera cuenta es admin).
4. En el panel lateral → **Settings** → **Models**, verifica que aparezcan los modelos de Ollama o añade tu `OPENAI_API_KEY`.
5. Inicia un nuevo chat y selecciona el modelo deseado.

## 💡 Casos de uso

- 🏠 **Chat privado familiar**: Instala en un mini-PC/NAS y usa modelos locales (llama3.2, phi3, gemma2) sin coste de API ni salida de datos.
- 🛠️ **Desarrollo asistido por IA**: Conecta Open WebUI a Ollama con `codellama` o `deepseek-coder` para autocompletado y refactorización offline.
- 📚 **RAG local**: Sube PDFs/Markdown a Open WebUI y haz preguntas sobre tus documentos usando embeddings locales.
- 🧪 **Pruebas de modelos**: Cambia entre `:main`, `:cuda`, `:ollama` según hardware; compara respuestas de modelos locales vs. cloud.
- 🔐 **Entornos air-gapped**: Usa `HF_HUB_OFFLINE=1` y modelos pre-cargados en Ollama para aislamiento total.

## 🔒 Acceso remoto seguro

> **⚠️ No expongas el puerto 3000 directamente a Internet.**

Opciones recomendadas:
- **Tailscale / WireGuard / ZeroTier**: Acceso VPN a tu red local.
- **Reverse Proxy + Authelia / Authelia / Pocket-ID**: HTTPS + SSO + 2FA (Traefik, Nginx Proxy Manager, Caddy).
- **Cloudflare Tunnel**: `cloudflared tunnel run --url http://localhost:3000` (gratis, sin abrir puertos).

## 🛠️ Gestión y mantenimiento

### Ver logs en tiempo real
```bash
docker logs -f open-webui
```

### Actualizar a la última versión
```bash
docker pull ghcr.io/open-webui/open-webui:main
docker rm -f open-webui
# Vuelve a ejecutar el mismo docker run / docker compose up -d
# Los datos persisten en el volumen
```

### Copia de seguridad del volumen
```bash
# Crea un tar del volumen open-webui en el directorio actual
docker run --rm -v open-webui:/data -v "$PWD":/backup alpine \
  tar czf /backup/open-webui-backup.tar.gz -C /data .
```
> **Importante**: Guarda el `.tar.gz` fuera del servidor (otro disco, NAS, cloud cifrado).

### Restaurar backup
```bash
docker run --rm -v open-webui:/data -v "$PWD":/backup alpine \
  tar xzf /backup/open-webui-backup.tar.gz -C /data
```

## 📝 Licencia

Open WebUI se distribuye bajo la **Open WebUI License** (licencia propia del proyecto, no estándar como MIT/GPL).  
👉 [Leer licencia completa](https://github.com/open-webui/open-webui/blob/main/LICENSE) antes de redistribuir o usar en producto comercial.

---

> 📖 **Guía completa en el blog**: [Cómo instalar Open WebUI en Docker paso a paso](https://genbyte.blogspot.com/2026/11/como-instalar-open-webui-en-docker-paso.html)  
> 🎥 **Vídeo tutorial**: [Canal GENBYTE en YouTube](https://www.youtube.com/@genbyte)  
> ☕ **Apoya el proyecto**: [Ko-fi](https://ko-fi.com/genbyte) | [Newsletter](https://genbyte.blogspot.com/newsletter)