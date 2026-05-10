# Bike Mechanic AI — Chatbot RAG

Chatbot especializado en **mecánica y mantenimiento de bicicletas**, construido con RAG (Retrieval-Augmented Generation). Combina una base de conocimiento local (33 manuales PDF de Shimano, SRAM, Park Tool y RockShox) con un LLM local vía Ollama para respuestas técnicas precisas sin depender de Internet ni de APIs de pago.

---

## Arquitectura

```
Usuario → Frontend (HTML + TailwindCSS)
               ↓  POST /chat
          Backend FastAPI
               ↓  búsqueda semántica (BAAI/bge-m3)
          ChromaDB  ←── ingestion.py ←── data/pdfs/*.pdf
               ↓  contexto relevante
          Ollama (llama3.2) ──→ Respuesta en español
```

| Componente | Tecnología |
|------------|-----------|
| LLM local | llama3.2 vía Ollama |
| Embeddings | BAAI/bge-m3 (multilingüe) |
| Vector store | ChromaDB |
| Backend | FastAPI + Python 3.11 |
| Frontend | HTML + TailwindCSS (CDN) |

---

## Opción A — Ejecución con Docker (recomendada)

> Requiere **Docker Desktop** instalado y en ejecución.

```bash
# 1. Clonar el repositorio
git clone <URL_DEL_REPO>
cd chatbot

# 2. Construir y levantar todos los servicios
docker compose up --build
```

El sistema hará automáticamente al primer arranque:
1. Esperar a que Ollama esté listo
2. Descargar el modelo `llama3.2` (~2 GB)
3. Generar embeddings con `BAAI/bge-m3` e indexar los 33 PDFs en ChromaDB (~20-30 min)
4. Levantar el servidor FastAPI en `http://localhost:8000`

> **En los reinicios posteriores** los pasos 2-3 se omiten gracias a los volúmenes de Docker.

### Abrir el frontend

Una vez que el servidor esté arriba, abre el archivo directamente en tu navegador:

```
frontend/index.html
```

### Detener los servicios

```bash
docker compose down
```

Para eliminar también los datos descargados (modelos, ChromaDB, embeddings):

```bash
docker compose down -v
```

---

## Opción B — Ejecución local (sin Docker)

### Prerequisitos

| Herramienta | Versión mínima | Instalación |
|-------------|---------------|-------------|
| Python | 3.10+ | [python.org](https://python.org) |
| Ollama | 0.3+ | [ollama.com](https://ollama.com) |

### Instalación

```bash
# 1. Crear entorno virtual
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux / Mac

# 2. Instalar dependencias
pip install -r requirements.txt
pip install langchain-huggingface cryptography

# 3. Descargar el modelo LLM
ollama pull llama3.2
```

### Configuración inicial (solo la primera vez)

```bash
# Indexar los PDFs en ChromaDB (tarda ~20-30 min por BAAI/bge-m3)
python backend/ingestion.py
```

### Ejecutar

```bash
# Terminal 1 — Servidor Ollama (si no corre como servicio)
ollama serve

# Terminal 2 — Backend FastAPI
python backend/main.py
# API disponible en http://localhost:8000
# Documentación interactiva: http://localhost:8000/docs
```

Luego abre `frontend/index.html` en tu navegador.

---

## Evaluación automática

Con el backend en ejecución, lanza el benchmark de 10 preguntas de dominio:

```bash
python eval/test_bench.py
# Resultados en eval/results.json
```

Resultado actual: **10/10 preguntas exitosas**, latencia promedio ~38 s/pregunta.

---

## Estructura del proyecto

```
chatbot/
├── backend/
│   ├── ingestion.py       # Pipeline: carga PDF → fragmenta → embeddings → ChromaDB
│   └── main.py            # Servidor FastAPI + cadena RAG
├── frontend/
│   └── index.html         # Interfaz de chat (sin dependencias de build)
├── data/
│   └── pdfs/              # 33 manuales PDF (Shimano, SRAM, Park Tool, RockShox)
├── eval/
│   ├── test_bench.py      # Benchmark automático (10 preguntas)
│   ├── results.json       # Resultados del último benchmark
│   └── Informe_Pruebas.md # Análisis de resultados
├── Dockerfile             # Imagen del backend + ingestión
├── docker-compose.yml     # Orquestación: ollama + api
├── entrypoint.sh          # Script de arranque del contenedor api
├── requirements.txt
└── README.md
```

---

## Parámetros configurables

### `backend/ingestion.py`
| Variable | Valor actual | Descripción |
|----------|-------------|-------------|
| `CHUNK_SIZE` | 512 | Tamaño de fragmentos en caracteres |
| `CHUNK_OVERLAP` | 100 | Solapamiento entre fragmentos |
| `EMBEDDING_MODEL` | `BAAI/bge-m3` | Modelo de embeddings multilingüe |

### `backend/main.py`
| Variable | Valor actual | Descripción |
|----------|-------------|-------------|
| `OLLAMA_MODEL` | `llama3.2` | Modelo LLM local |
| `TOP_K` | 8 | Chunks recuperados por consulta |
| `temperature` | 0.2 | Creatividad del LLM (0 = determinista) |

---

## Cambiar de modelo

```bash
# Descargar otro modelo
ollama pull deepseek-r1:7b

# Cambiar en backend/main.py:
# OLLAMA_MODEL = "deepseek-r1:7b"
```

Para Docker, modifica también la variable en `entrypoint.sh`:
```bash
MODEL="deepseek-r1:7b"
```

---

## Notas técnicas

- El frontend se conecta a `http://localhost:8000` — no requiere servidor web, solo abrir el HTML.
- ChromaDB persiste en `data/chroma_db/` (local) o en el volumen `chroma_db` (Docker).
- El historial de conversación se mantiene en el navegador (últimas 6 interacciones enviadas al LLM).
- Cada pregunta del benchmark se evalúa de forma independiente (sin historial compartido).
