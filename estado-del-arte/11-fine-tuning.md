# 11. Fine-tuning: adaptar los modelos a nuestros datos

Los modelos preentrenados (segmentación, embeddings, Whisper) se entrenaron sobre todo con audio en inglés y chino, y con ciertos tipos de grabación. Cuando nuestros audios son distintos (español, otro tipo de micrófono, otro dominio), el rendimiento baja. El **fine-tuning** (ajuste fino) continúa el entrenamiento de un modelo preentrenado con pocos datos propios para adaptarlo.

Es la forma más barata de mejorar: no hace falta entrenar desde cero ni grandes cantidades de datos.

## 11.1 Qué se puede ajustar en nuestro pipeline

| Componente | Qué se ajusta | Datos necesarios | Costo | Impacto esperado |
|------------|---------------|------------------|-------|------------------|
| **Hiperparámetros del pipeline** | `δ` (umbral del clustering), `min_cluster_size`, duraciones mínimas | Unos pocos audios con referencia RTTM | Muy bajo (no se entrena ninguna red) | Medio; es lo primero que conviene hacer |
| **Modelo de segmentación** (Fase 1) | Pesos de PyanNet | Horas de audio anotado en RTTM del dominio | Bajo–medio (red pequeña, cabe en una GPU modesta) | **Alto**; es lo que más mejora el DER según la receta de pyannote |
| **Modelo de embeddings** (Fase 2) | Pesos de ResNet34 | Muchos hablantes distintos etiquetados | Alto | Medio; rara vez compensa en un proyecto de curso |
| **Whisper** (ASR) | Pesos del encoder-decoder | Audio + transcripción del dominio | Medio–alto | Útil si el vocabulario o el acento son muy específicos |
| **Modelos de atributos** (AVA) | Sondas o modelos SSL | Audio con metadatos (edad, sexo, acento) | Bajo (sondas) a medio (wav2vec 2.0) | Es la única vía con resultados sólidos en atributos |

## 11.2 La receta de pyannote: el enfoque principal

El paper central del proyecto, [Bredin (2023)](https://www.isca-archive.org/interspeech_2023/bredin23_interspeech.html), no solo describe el pipeline: propone una **receta** para que cualquiera lo adapte a sus datos. Tiene dos pasos:

1. **Afinar el modelo de segmentación** con audio anotado del dominio. Se parte de los pesos preentrenados y se entrena unas pocas épocas con nuestros datos.
2. **Optimizar los hiperparámetros del pipeline** (umbral de clustering, duraciones mínimas) sobre un conjunto de desarrollo, buscando el menor DER. pyannote incluye un optimizador para esto.

En el paper, esta receta mejora el DER de forma considerable en cada dominio respecto al modelo sin adaptar, y fue la base de sus resultados en retos (1.º en Ego4D 2022 y Albayzin 2022).

**Tutorial oficial:** [`tutorials/adapting_pretrained_pipeline.ipynb`](https://github.com/pyannote/pyannote-audio/blob/develop/tutorials/adapting_pretrained_pipeline.ipynb) en el repositorio de pyannote. Hay también tutoriales de entrenamiento (`training_a_model.ipynb`) y de optimización (`multi_objective_optimization.ipynb`).

**Evidencia en español:** en el reto Albayzin 2024 (diarización de televisión española, RTVE) participaron equipos que afinaron modelos de segmentación de pyannote con datos en español.

### Por qué encaja con DASR

Como DASR reproduce el pipeline módulo a módulo, el fine-tuning se puede aplicar **por partes** y medir el efecto de cada una:
- primero ajustar solo `δ` y comparar el DER;
- después afinar la segmentación y comparar de nuevo.

Cada mejora queda atribuida a un cambio concreto, que es justo lo que se busca en un trabajo de curso.

## 11.3 Fine-tuning en el estado del arte reciente

La tendencia 2024–2026 es afinar **modelos autosupervisados grandes** (WavLM, wav2vec 2.0) para diarización, en lugar de entrenar redes pequeñas desde cero:

- **DiariZen** ([Han et al., 2024](https://arxiv.org/abs/2409.09408); [2025](https://arxiv.org/abs/2506.18623)) reemplaza SincNet + LSTM por **WavLM afinado** + Conformer, manteniendo la estructura local + global de pyannote. Es hoy el mejor sistema abierto en varios benchmarks. Su segundo paper **poda** WavLM para que sea más ligero sin perder precisión.
- pyannote 4.0 incorporó el **clustering VBx** de ese mismo grupo (BUT).

Ambos confirman que la estructura que reproducimos es la base sobre la que se construyen las mejoras actuales: **las mejoras recientes cambian piezas, no la arquitectura**.

## 11.4 Recomendación para el proyecto

1. **Línea base:** reproducir el pipeline 3.1 con los modelos preentrenados, sin ajustar nada.
2. **Ajuste de hiperparámetros:** optimizar `δ` y `min_cluster_size` con unos pocos audios propios anotados.
3. **(Opcional) Afinar la segmentación** con el tutorial de pyannote si hay datos anotados y GPU.
4. **AVA:** entrenar sondas sobre los embeddings con Common Voice; afinar wav2vec 2.0 solo si las sondas no bastan.

Reportar el DER en cada paso permite mostrar cuánto aporta cada ajuste.
