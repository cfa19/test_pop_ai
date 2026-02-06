# Resultados de Evaluacion v3 - Intent Classifier Models

Evaluacion de 2 modelos fine-tuned para clasificacion de intents con **8 categorias** (7 intents + chitchat). off_topic se maneja por semantic gate.

**Dataset de test:** 1000 ejemplos (shuffle aleatorio, seed=42) de `training/data/processed/`

---

## Modelos evaluados

| Modelo | Nombre base (HuggingFace) | Arquitectura | Ruta local |
|--------|--------------------------|--------------|------------|
| **BAAI** | `BAAI/bge-small-en-v1.5` | BERT 12 capas | `training/scripts/models/BAAI/final` |
| **all-MiniLM-L6-v2** | `sentence-transformers/all-MiniLM-L6-v2` | BERT 6 capas | `training/scripts/models/all-MiniLM-L6-v2/final` |

---

## 1. PRECISION Y METRICAS GLOBALES

| Metrica | BAAI | all-MiniLM-L6-v2 | Diferencia |
|---------|------|-------------------|------------|
| **Accuracy** | **99.50%** | 98.70% | BAAI +0.80% |
| **Precision** | **99.51%** | 98.70% | BAAI +0.81% |
| **Recall** | **99.50%** | 98.70% | BAAI +0.80% |
| **F1 Score** | **99.50%** | 98.70% | BAAI +0.80% |
| **Errores totales** | **5** | 13 | BAAI 60% menos errores |

### Interpretacion
- BAAI comete **5 errores** en 1000 mensajes (0.5% error rate)
- all-MiniLM comete **13 errores** en 1000 mensajes (1.3% error rate)
- BAAI tiene **2.6x menos errores** que all-MiniLM

---

## 2. CONFIDENCE (CONFIANZA DEL MODELO)

| Metrica | BAAI | all-MiniLM-L6-v2 | Mejor |
|---------|------|-------------------|-------|
| **Confidence promedio** | 91.58% | **94.86%** | all-MiniLM |
| **Confidence en aciertos** | ~92% | ~95% | all-MiniLM |

### Interpretacion
- all-MiniLM tiene **mayor confianza** en sus predicciones
- BAAI es mas "conservador" pero **mas preciso**
- Mayor confidence no siempre = mejor modelo (all-MiniLM tiene mas errores)

---

## 3. LATENCIA Y VELOCIDAD

| Metrica | BAAI | all-MiniLM-L6-v2 | Diferencia |
|---------|------|-------------------|------------|
| **Latencia por mensaje** | 9.65ms | **6.47ms** | all-MiniLM 1.5x mas rapido |
| **Mensajes por segundo** | ~104 msg/s | **~155 msg/s** | all-MiniLM +49% throughput |
| **Tiempo para 1000 mensajes** | 9.65s | **6.47s** | all-MiniLM ahorra 3.18s |

### Interpretacion
- all-MiniLM es **49% mas rapido** que BAAI
- En produccion con alto trafico, all-MiniLM puede manejar **50% mas requests**
- La diferencia de 3.18ms por mensaje es **imperceptible** para el usuario

---

## 4. RECURSOS Y MEMORIA

| Metrica | BAAI | all-MiniLM-L6-v2 | Diferencia |
|---------|------|-------------------|------------|
| **Tamaño en disco** | 133 MB | **90 MB** | all-MiniLM 1.5x menor |
| **Capas del modelo** | 12 | **6** | all-MiniLM 2x menos capas |
| **Hidden size** | 384 | 384 | Igual |
| **RAM estimada** | ~400 MB | **~250 MB** | all-MiniLM 1.6x menor |
| **Parametros** | ~33M | **~22M** | all-MiniLM 1.5x menos |

### Interpretacion
- all-MiniLM usa **37% menos RAM** y **32% menos disco**
- Para servers con recursos limitados (t3.micro, etc), all-MiniLM es mejor opcion
- BAAI requiere GPU con mas VRAM para batch processing

---

## 5. PRECISION POR CATEGORIA (F1-Score)

| Categoria | BAAI | all-MiniLM-L6-v2 | Mejor | Diferencia |
|-----------|------|-------------------|-------|------------|
| rag_query | **1.00** | **1.00** | Empate | 0.00 |
| chitchat | **1.00** | **1.00** | Empate | 0.00 |
| emotional | 0.99 | **1.00** | all-MiniLM | +0.01 |
| learning | **1.00** | 0.98 | BAAI | +0.02 |
| psychological | **1.00** | 0.99 | BAAI | +0.01 |
| professional | **0.99** | 0.97 | BAAI | +0.02 |
| aspirational | **0.99** | 0.97 | BAAI | +0.02 |
| social | **0.99** | 0.98 | BAAI | +0.01 |

### Analisis detallado

**Categorias perfectas (1.00 F1) en ambos:**
- `rag_query` - Preguntas de informacion
- `chitchat` - Saludos y conversacion casual

**Categorias donde BAAI es mejor:**
- `professional` (+0.02) - Habilidades y experiencia laboral
- `aspirational` (+0.02) - Metas y suenos de carrera
- `learning` (+0.02) - Preferencias de aprendizaje
- `psychological` (+0.01) - Personalidad y valores
- `social` (+0.01) - Red profesional

**Categorias donde all-MiniLM es mejor:**
- `emotional` (+0.01) - Bienestar emocional

### Categoria critica: Professional
La categoria `professional` es la mas dificil (mas confusiones):
- BAAI: 0.99 F1 (2 errores)
- all-MiniLM: 0.97 F1 (5 errores)
- **BAAI es 2.5x mejor** en esta categoria critica

---

## 6. ANALISIS DE ERRORES

### BAAI (5 errores en 1000)

| Error | Categoria real | Prediccion | Tipo de error |
|-------|---------------|------------|---------------|
| 1 | professional | aspirational | Confusion de contexto |
| 2 | professional | learning | Confusion de contexto |
| 3 | social | emotional | Confusion emocional |
| 4 | social | professional | Confusion de contexto |
| 5 | aspirational | professional | Confusion de contexto |

### all-MiniLM-L6-v2 (13 errores en 1000)

| Categoria | Errores | Patron |
|-----------|---------|--------|
| professional | 5 | Confunde con aspirational/learning |
| learning | 3 | Confunde con professional |
| social | 2 | Confunde con emotional |
| aspirational | 2 | Confunde con professional |
| psychological | 1 | Confunde con emotional |

### Patrones de error comunes
1. **professional ↔ aspirational**: Mensajes sobre metas profesionales
2. **professional ↔ learning**: Mensajes sobre aprender habilidades
3. **social ↔ emotional**: Mensajes sobre relaciones con carga emocional

---

## 7. RECOMENDACION PARA PRODUCCION

### Tabla de decision

| Escenario | Modelo recomendado | Razon |
|-----------|-------------------|-------|
| **Maxima precision** | **BAAI** | 60% menos errores, mejor en categorias criticas |
| **Recursos limitados** | all-MiniLM | 40% menos RAM, 30% menos disco |
| **Alto trafico (>1000 req/min)** | all-MiniLM | 50% mas throughput |
| **Startup/MVP** | all-MiniLM | Menor costo de infraestructura |
| **Enterprise/precision critica** | **BAAI** | Menos errores = mejor UX |
| **Balance general** | **BAAI** | Diferencia de recursos es aceptable |

### Nuestra recomendacion: **BAAI**

**Por que BAAI:**
1. **60% menos errores** (5 vs 13) - Mejor experiencia de usuario
2. **Mejor en professional** - Categoria critica para career coaching
3. **La diferencia de latencia (3ms) es imperceptible** para el usuario
4. **El costo adicional de recursos es minimo** (~150MB mas de RAM)

**Cuando elegir all-MiniLM:**
- Server con menos de 512MB RAM disponible
- Necesidad de procesar >150 mensajes/segundo
- Presupuesto muy limitado de infraestructura

---

## 8. CONFIGURACION RECOMENDADA

### Para usar BAAI (recomendado):

```bash
# .env
INTENT_CLASSIFIER_TYPE=BAAI
INTENT_CLASSIFIER_MODEL_PATH=training/scripts/models/BAAI/final
```

### Para usar all-MiniLM-L6-v2:

```bash
# .env
INTENT_CLASSIFIER_TYPE=all-MiniLM-L6-v2
INTENT_CLASSIFIER_MODEL_PATH=training/scripts/models/all-MiniLM-L6-v2/final
```

---

## 9. COMANDOS DE EVALUACION

```bash
# Evaluar BAAI
python training/scripts/evaluate.py \
  --task intent \
  --model training/scripts/models/BAAI/final \
  --test-data training/data/processed \
  --output training/scripts/eval_results/baai_test

# Evaluar all-MiniLM-L6-v2
python training/scripts/evaluate.py \
  --task intent \
  --model training/scripts/models/all-MiniLM-L6-v2/final \
  --test-data training/data/processed \
  --output training/scripts/eval_results/all-MiniLM-L6-v2
```

---

## 10. RESUMEN EJECUTIVO

| | BAAI | all-MiniLM-L6-v2 |
|---|:---:|:---:|
| **Accuracy** | **99.50%** | 98.70% |
| **Errores/1000** | **5** | 13 |
| **Latencia** | 9.65ms | **6.47ms** |
| **RAM** | 400MB | **250MB** |
| **Disco** | 133MB | **90MB** |
| **Professional F1** | **0.99** | 0.97 |

### Veredicto final

**BAAI gana** en precision y categorias criticas.
**all-MiniLM gana** en velocidad y eficiencia de recursos.

Para un **career coaching chatbot** donde la precision importa mas que milisegundos de latencia, **BAAI es la mejor opcion**.
