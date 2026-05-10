# Informe de Pruebas — Chatbot RAG: Mecánica de Bicicletas

**Fecha de evaluación:** _completar_
**Modelo LLM:** llama3.2 (Ollama local)
**Modelo de embeddings:** all-MiniLM-L6-v2
**Vector Store:** ChromaDB
**Documentos fuente:** _listar PDFs/MD utilizados_

---

## 1. Tabla de Resultados

| # | Pregunta (resumen) | Respuesta correcta | Fuente citada | Latencia (s) | Calificación (1-5) |
|---|---|---|---|---|---|
| 1 | Purga frenos Shimano hidráulicos | ☐ Sí / ☐ No / ☐ Parcial | | | |
| 2 | Tornillos H y L desviador delantero | ☐ Sí / ☐ No / ☐ Parcial | | | |
| 3 | Extracción cassette 11v + torque | ☐ Sí / ☐ No / ☐ Parcial | | | |
| 4 | PSI horquilla RockShox Lyrik 85 kg | ☐ Sí / ☐ No / ☐ Parcial | | | |
| 5 | Diagnóstico crujido headset ruta | ☐ Sí / ☐ No / ☐ Parcial | | | |
| 6 | Caída vs subida cambio indexado | ☐ Sí / ☐ No / ☐ Parcial | | | |
| 7 | Desgaste cadena + chain checker | ☐ Sí / ☐ No / ☐ Parcial | | | |
| 8 | Reemplazo cámara rueda trasera | ☐ Sí / ☐ No / ☐ Parcial | | | |
| 9 | Lubricante seco vs húmedo | ☐ Sí / ☐ No / ☐ Parcial | | | |
| 10 | Pedalier Hollowtech II sentido rosca | ☐ Sí / ☐ No / ☐ Parcial | | | |

**Promedio calificación:** _/5_ | **Tasa de éxito:** _/10_ | **Latencia promedio:** _s_

---

## 2. Análisis de Resultados

### 2.1 Preguntas bien respondidas
_Describir qué preguntas tuvieron respuestas completas y por qué (cobertura en documentos fuente, chunks relevantes recuperados, etc.)_

### 2.2 Preguntas con respuesta parcial
_Indicar qué información faltó y si el modelo halució o reconoció su limitación._

### 2.3 Preguntas sin respuesta satisfactoria
_Identificar las preguntas fallidas y la causa raíz (falta de fuentes, chunk demasiado pequeño/grande, limitación del modelo, etc.)_

---

## 3. Limitaciones Identificadas

### 3.1 Limitaciones de los documentos fuente
- La calidad de las respuestas depende directamente de los PDFs/Markdown disponibles.
- Documentos escaneados sin OCR producen chunks vacíos o ilegibles.
- Información técnica muy específica (torques, PSI por modelo) requiere manuales de fabricante exactos.

### 3.2 Limitaciones del pipeline RAG
- **Granularidad de chunks:** Chunks muy grandes diluyen la información relevante; chunks muy pequeños pierden contexto.
- **Top-K fijo:** Con K=5 puede recuperarse contexto irrelevante si los embeddings no discriminan bien términos técnicos similares.
- **Sin reranking:** No hay una etapa de reordenamiento semántico posterior a la búsqueda vectorial.

### 3.3 Limitaciones del modelo LLM
- llama3.2 (3B) puede alucinar valores numéricos (torques, PSI) no presentes en el contexto.
- Las respuestas en español pueden ser menos fluidas que en inglés para modelos no ajustados en español.
- Latencia de ~15-60s por pregunta en CPU es aceptable para prototipo pero no para producción.

---

## 4. Propuestas de Mejora

| Prioridad | Mejora | Impacto esperado |
|-----------|--------|-----------------|
| Alta | Agregar más documentos fuente (Shimano, SRAM, Park Tool completos) | +30% cobertura de preguntas |
| Alta | Implementar reranking con cross-encoder (`ms-marco-MiniLM-L-6-v2`) | Reducir contexto irrelevante |
| Media | Ajustar `CHUNK_SIZE` a 600 con overlap 200 para tablas de torque | Mejor recuperación de datos técnicos |
| Media | Migrar a `llama3.2:11b` o `deepseek-r1:14b` si hay GPU disponible | Mayor precisión en respuestas |
| Media | Añadir metadata filtering por categoría (frenos, transmisión, etc.) | Búsqueda más precisa |
| Baja | Implementar HyDE (Hypothetical Document Embeddings) para queries ambiguas | Mejora en preguntas complejas |
| Baja | Agregar evaluación automática con LLM-as-judge (GPT-4 o Claude) | Métricas objetivas de calidad |

---

## 5. Métricas Técnicas del Sistema

| Métrica | Valor |
|---------|-------|
| Total documentos fuente | |
| Total chunks generados | |
| Dimensión de embeddings (MiniLM-L6) | 384 |
| Tiempo de ingestión total | |
| Tiempo promedio por consulta | |
| RAM usada por ChromaDB | |
| RAM usada por Ollama (llama3.2) | ~2 GB |

---

## 6. Conclusión

_Resumen de 2-3 párrafos sobre el desempeño general del sistema, si cumple con los objetivos de la Fase 2, y los próximos pasos recomendados para la Fase 3._
