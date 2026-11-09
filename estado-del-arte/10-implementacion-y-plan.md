# 10. Implementación y plan de trabajo

## 10.1 Del repositorio de pyannote a los módulos de DASR

El código de referencia está en [github.com/pyannote/pyannote-audio](https://github.com/pyannote/pyannote-audio). Las rutas siguientes son de la **versión 3.1.1** (etiqueta `3.1.1`), que es la que reproducimos.

| Módulo DASR | Qué implementa | Referencia en pyannote.audio |
|-------------|----------------|------------------------------|
| `Core/` | `AudioSignal`: audio mono 16 kHz, tipos compartidos | `core/io.py` → `Audio`; librería `pyannote.core` (`Segment`, `SlidingWindowFeature`) |
| `Segmentation/` | `local_segments` + `binary_segments` (Fase 1) | `pipelines/speaker_diarization.py` → `get_segmentations()`; `utils/powerset.py` |
| `ASR/` (VAD, FA) | Voz a partir de la segmentación; Whisper + alineación forzada | — (externo: Whisper, wav2vec 2.0 CTC; referencia: WhisperX) |
| `D/LSE/` | Embeddings con máscara sin solapamiento (Fase 2) | `speaker_diarization.py` → `get_embeddings(exclude_overlap=True)` |
| `D/GAC/` | AHC con centroides + `δ` + `min_cluster_size` + húngaro (Fase 3) | `pipelines/clustering.py` → `AgglomerativeClustering`, `assign_embeddings()` |
| `D/build_turns` | Conteo `Kf`, top-`Kf`, frames → turnos | `pipelines/utils/diarization.py` → `speaker_count()`, `to_diarization()`; `speaker_diarization.py` → `reconstruct()` |
| `Pipeline/` | `assign_speakers`: palabra ↔ hablante | — (propio) |
| `AVA/` | Atributos vocales potenciales | — (propio) |

### Qué hace cada función de pyannote

- **`get_segmentations()`** aplica el modelo de segmentación en ventana deslizante (nuestra Fase 1). El paso lo controla `segmentation_step = 0.1` (10 % de la ventana).
- **`get_embeddings(exclude_overlap=True)`** extrae un embedding por (ventana, hablante local). Con `exclude_overlap` pone a cero los frames donde la suma de `b` es ≥ 2 (solapamiento) y, si la voz limpia es demasiado corta, usa la máscara completa.
- **`AgglomerativeClustering.cluster()`** normaliza los embeddings, llama a `scipy.cluster.hierarchy.linkage(..., method="centroid", metric="euclidean")`, corta con `fcluster(..., threshold, criterion="distance")` y reasigna los clústeres pequeños.
- **`assign_embeddings()`** hace la asignación restringida por ventana con `scipy.optimize.linear_sum_assignment` (algoritmo húngaro).
- **`speaker_count()`** recorta el 10 % de cada borde de las ventanas, suma la segmentación binaria, promedia entre ventanas y redondea: es `Kf`.
- **`reconstruct()`** + **`to_diarization()`** construyen la puntuación por clúster y eligen los top-`Kf` hablantes por frame.

> **Idea clave:** fijar la **versión 3.1** como referencia. En la 4.0 (*community-1*) el clustering por defecto pasa a ser **VBx** (`clustering="VBxClustering"` en el código actual), así que comparar contra la 4.0 mezclaría dos algoritmos distintos.

## 10.2 Plan de reproducción paso a paso (a menor escala)

| # | Etapa | Qué hacer | Cómo se valida |
|---|-------|-----------|----------------|
| 1 | **Datos** | Subconjunto pequeño de un corpus con referencia (por ejemplo AMI) + grabaciones propias anotadas en RTTM | Las referencias se cargan y se visualizan bien |
| 2 | **Fase 1** | Segmentación con `segmentation-3.0` en ventana deslizante + binarización | Nuestros tensores `s` y `b` coinciden con los de pyannote |
| 3 | **Fase 2** | LSE: máscara limpia + WeSpeaker ResNet34 | Nuestros `e(w,k)` coinciden con los de pyannote |
| 4 | **Fase 3** | GAC propio con SciPy + reconstrucción | Mismas etiquetas de clúster; **DER** comparable al de la tabla 3.1 |
| 5 | **ASR + fusión** | Whisper + alineación forzada; `assign_speakers` | **WER**; revisión manual de la transcripción por hablante |
| 6 | **AVA** | Sondas sobre embeddings y rasgos | **UAR / MAE** con particiones por hablante |

### Criterio en cada etapa

Con el **mismo audio**, nuestra salida intermedia debe **coincidir** con la de pyannote: tensores de segmentación, embeddings, etiquetas de clúster. **Solo se avanza a la siguiente etapa cuando la anterior coincide.** Los tests de `test_wiring.py` verifican que las piezas encajan entre sí.

Esto convierte un proyecto grande en pasos pequeños verificables: si algo falla en la Fase 3, sabemos que las Fases 1 y 2 están bien.

> **Idea clave:** reproducir no es usar la librería: es **poder explicar y verificar cada tensor intermedio**.

## 10.3 En síntesis

1. **Fase 1, preparación:** la red ve 10 s, detecta hablantes y solapamientos; la binarización decide quién habla en cada frame.
2. **Fase 2, procesamiento local:** un embedding por hablante local, con toda su voz limpia de la ventana.
3. **Fase 3, procesamiento global:** AHC con centroides y umbral `δ` da identidades globales coherentes; la reconstrucción cuenta y elige.
4. **Integración:** el tiempo une texto y hablantes; los embeddings abren el análisis de atributos vocales.

La diarización de pyannote combina lo mejor de dos mundos: una red neuronal local que entiende solapamientos y un clustering global que da coherencia a todo el audio. **La contribución del proyecto** es reproducir ese pipeline módulo a módulo en DASR, integrarlo con el ASR mediante el tiempo y aprovechar los embeddings de voz para estimar atributos vocales potenciales.

**Próximo paso:** completar la Fase 1 y validar sus tensores contra pyannote.
