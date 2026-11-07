# 4. Embeddings de hablante e identificación

Los embeddings son el puente entre la diarización (agrupar voces) y la identificación (ponerles nombre).

## 4.1 Evolución de los embeddings

| Generación | Modelo | Idea |
|------------|--------|------|
| Estadística | i-vectors (2011) | Factor analysis sobre GMM-UBM. Superado, pero aparece en la bibliografía clásica. |
| Neuronal | **x-vectors** (Snyder et al., 2018) | TDNN + *statistics pooling*; fue el estándar de facto. |
| Neuronal | **ECAPA-TDNN** ([Desplanques et al., 2020](https://arxiv.org/abs/2005.07143)) | TDNN con atención por canal (SE), agregación multicapa y *attentive pooling*. El modelo más usado 2020–2026; viene listo en SpeechBrain. |
| Neuronal | **ResNet-34 / ResNet-221** (WeSpeaker) | CNN 2D sobre fbanks. ResNet34 es el embedding de pyannote 3.x. |
| Neuronal | **CAM++** ([Wang et al., 2023](https://arxiv.org/abs/2303.00332)) | D-TDNN con *context-aware masking*: ~0.73% de EER en VoxCeleb1-O con la mitad de parámetros que ECAPA. De 3D-Speaker (Alibaba). |
| Neuronal | **ReDimNet** ([Yakovlev et al., 2024](https://arxiv.org/abs/2407.18223)) | Remodela dimensiones entre bloques 1D y 2D; de los mejores resultados en VoxCeleb con pocos parámetros. |
| Autosupervisado | **WavLM** ([Chen et al., 2021](https://arxiv.org/abs/2110.13900)) + ECAPA | Representaciones de WavLM-Large como entrada a un ECAPA: ~0.38% de EER en VoxCeleb1-O. Es lo más preciso, pero pesado. |

**Pérdidas de entrenamiento:** casi todos se entrenan como clasificadores de miles de hablantes (VoxCeleb2) con márgenes angulares (**AAM-softmax / ArcFace**) y luego se descarta la capa de clasificación; el vector de la penúltima capa es el embedding.

**Puntuación:** similitud **coseno** entre embeddings, a veces con PLDA o calibración. Con buenos embeddings modernos, coseno + normalización de puntuaciones (*AS-norm*) es suficiente.

## 4.2 Identificación de hablantes (ponerle nombre a las voces)

Flujo típico, que es el que necesita el proyecto:

1. **Registro** (*enrollment*): para cada persona conocida, 10–30 s de voz limpia → embedding promedio (perfil de voz).
2. **Diarización** del audio nuevo → grupos `SPK_0`, `SPK_1`…
3. **Embedding de cada grupo** (promedio de sus segmentos más limpios, sin solapamiento).
4. **Comparación** coseno contra cada perfil:
   - si la mejor similitud supera un **umbral** → se asigna ese nombre;
   - si no → **"desconocido"** (conjunto abierto).
5. Opcional: asignación global con el **algoritmo húngaro** para que dos grupos no reciban el mismo nombre.

Puntos delicados:
- **Calibrar el umbral** con datos propios: depende del modelo, del canal (teléfono, micrófono, video) y del idioma.
- **Desajuste de dominio:** los modelos se entrenan sobre todo con VoxCeleb (entrevistas de famosos, en su mayoría en inglés). El rendimiento cae con otro idioma, otro canal o audio ruidoso; conviene evaluar con voces en español.
- **Segmentos cortos (< 2 s)** dan embeddings poco fiables: mejor agregar varios.

## 4.3 Benchmarks de verificación

- **VoxCeleb1-O / E / H** (EER, minDCF): el estándar. Los mejores sistemas están por debajo de 0.5% de EER en Vox1-O, así que el benchmark está casi saturado.
- **CN-Celeb:** más difícil (múltiples géneros, chino); los EER suben mucho (CAM++ ≈ 6.8%).
- **VoxSRC** (retos anuales 2019–2023): incluían una pista de diarización sobre VoxConverse.
- **Conjunto abierto:** [VoxWatch](https://arxiv.org/abs/2307.00169) propone un benchmark de identificación abierta sobre VoxCeleb, más cercano a la tarea real del proyecto.

## 4.4 Qué significa para el proyecto

- Para **identificar**, un ECAPA-TDNN (SpeechBrain) o un ResNet/CAM++ (WeSpeaker / 3D-Speaker) preentrenados bastan como base.
- El aporte de ML puede estar en **evaluar y calibrar** la identificación abierta con voces en español, o en **ajustar** (*fine-tuning*) los embeddings a ese dominio.
