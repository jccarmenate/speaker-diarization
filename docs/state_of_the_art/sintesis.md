# Estado del arte para DASR: diarización, ASR y atributos vocales

Punto de partida para quien trabaje en DASR (reproducción del pipeline de pyannote 3.1, fusión con el ASR y análisis de atributos vocales). Revisión hecha el 12 de noviembre de 2026. Este archivo es la síntesis: responde a las preguntas de partida, resume lo que condiciona el trabajo y propone una ruta de lectura. El detalle está en los cinco bancos de referencias de esta carpeta, donde cada trabajo tiene su referencia completa con enlace, de qué va, por qué importa para DASR y unas etiquetas de relevancia.

La explicación del método (las tres fases, la fusión y AVA) está en [`../fundamentals/`](../fundamentals/) y en la presentación de [`../presentation/`](../presentation/). Conviene leer antes al menos el [README del marco teórico](../fundamentals/README.md).

Esta carpeta también contiene el artículo del equipo (`state-of-the-art.tex` y su PDF) y su matriz por preguntas de investigación (`sota.xlsx`), descritos en el [README](README.md).

## Cómo usar esta carpeta

| Archivo | Frente | Entradas |
|---|---|---|
| [`1-diarizacion.md`](1-diarizacion.md) | Sistemas de diarización: pyannote y su enfoque local + global, EEND y derivados, la línea de NVIDIA (NeMo), DiariZen, revisiones, benchmarks y limitaciones. | 27 |
| [`2-representaciones-y-clustering.md`](2-representaciones-y-clustering.md) | Embeddings de hablante, modelos autosupervisados y algoritmos de clustering (AHC, espectral, VBx). | 22 |
| [`3-asr-y-atribucion.md`](3-asr-y-atribucion.md) | ASR con tiempos por palabra, alineación forzada, modelos conjuntos de ASR y diarización, y métricas de transcripción multi-hablante. | 17 |
| [`4-atributos-vocales.md`](4-atributos-vocales.md) | Qué guardan las representaciones, edad y sexo, acento, rasgos acústicos, paralingüística y sesgos. | 14 |
| [`5-evaluacion-y-datos.md`](5-evaluacion-y-datos.md) | Métricas de diarización, corpus de diarización y de hablantes, y datos en español. | 13 |

**Los PDF están en [`pdfs/`](pdfs/).** Copia local de los artículos de los cinco bancos para leerlos sin conexión, una subcarpeta por banco. [`pdfs/INDICE.md`](pdfs/INDICE.md) enlaza cada entrada con su archivo y lista al final los que hay que bajar a mano desde el navegador (descarga del 22 de noviembre de 2026: 79 de 93 entradas; 3 son páginas web sin PDF; de los 11 restantes, 8 son de pago y 3 son de acceso abierto pero sus editoriales bloquean las descargas automáticas).

**La matriz está en [`matriz-bancos-DASR.xlsx`](matriz-bancos-DASR.xlsx).** Cruza 16 criterios que pide DASR con 26 trabajos clave. La hoja *Codebook* explica cada criterio, *Evidencia* tiene una fila por cada marca para completar con la cita textual del PDF, y *Decisiones* reúne las 12 decisiones del proyecto con los trabajos que las apoyan y dónde están escritas. Las marcas salen de las fichas de los bancos, no de una lectura completa de cada PDF.

**Cómo leer cada entrada.** Todas tienen la misma forma: referencia (autores, año, título, venue y enlace), qué hace, relevancia para DASR, etiquetas y PDF. Las etiquetas cambian un poco según el banco, pero siempre incluyen la relevancia (alta, media o baja) y el código si existe. Relevancia alta significa "leer entero", media "leer el resumen y la sección del método", baja "saber que existe".

**Lo marcado "(no verificado)".** Autores, año, título y enlace de cada referencia se contrastaron con arXiv o Crossref. Lo que no se pudo confirmar (sobre todo venues que en arXiv solo constan como "enviado" y algunos repositorios de código) va marcado así. Antes de citar algo en el informe, comprobarlo en la fuente.

**Convención de citas en esta síntesis.** [B1 §2.4] es la sección 2.4 del banco 1.

## Respuesta corta a las preguntas de partida

### ¿Cuál es el estado del arte en diarización y dónde queda pyannote 3.1?

Los mejores sistemas abiertos siguen el esquema local + global que reproduce DASR: una red segmenta ventanas cortas y detecta solapamiento, y un clustering da identidad global. Así funcionan pyannote 3.1 y su sucesor *community-1* [B1 §1.1, §1.6], DiariZen [B1 §4.1], EEND-VC [B1 §2.4] y diart [B1 §1.5].

La alternativa es la línea end-to-end (EEND, Sortformer, Nemotron 3) [B1 §2, §3]: resuelve la permutación dentro de la red y limita el número de hablantes (4 en Sortformer, 8 en Nemotron 3).

En la comparación independiente más reciente, sobre 196.6 h en cinco idiomas incluido el español, el mejor sistema abierto es DiariZen con 13.3 % de DER [B1 §5.4]. DiariZen es precisamente pyannote con WavLM en la segmentación y VBx en el clustering. **La base de DASR no está desfasada: las mejoras recientes cambian piezas del pipeline, no su arquitectura.**

### ¿Hay un estado del arte con el que compararse?

Sí, y casi todo tiene código público.

| Baseline | Qué es | Dónde está | Por qué incluirlo |
|---|---|---|---|
| pyannote 3.1 (librería) | El pipeline oficial que DASR reproduce. | https://github.com/pyannote/pyannote-audio (etiqueta `3.1.1`) | Referencia de cada tensor intermedio y del DER final [B1 §1.1]. |
| pyannote *community-1* | pyannote 4.0: VBx y diarización exclusiva. | Hugging Face (acceso restringido) | Mide cuánto aporta VBx frente a AHC con centroides [B1 §1.6]. |
| DiariZen | WavLM + Conformer + VBx sobre la misma estructura. | https://github.com/BUTSpeechFIT/DiariZen | El mejor sistema abierto del benchmark de 2025 [B1 §4.1, §5.4]. |
| Streaming Sortformer / Nemotron 3 | Diarización end-to-end de NVIDIA. | https://github.com/NVIDIA-NeMo/Speech | Comparación externa con otro paradigma [B1 §3.3, §3.4]. |
| WhisperX | Whisper + alineación forzada + diarización de pyannote. | https://github.com/m-bain/whisperX | Referencia de la rama ASR y de la fusión [B3 §1.3]. |
| Pipeline clásico | Embeddings por segmento fijo (x-vector o ECAPA) y AHC, sin segmentación local. | SpeechBrain o WeSpeaker | Muestra cuánto aportan las Fases 1 y 2 de pyannote [B2 §1.2, §1.6, §3.1]. |
| Diarización de referencia | Usar la RTTM verdadera en la fusión. | Datos anotados | Aísla el error del ASR del error de diarización [B3 §3.2]. |
| Modelos de atributos publicados | wav2vec 2.0 de audEERING y WavLM de Yang et al. | Hugging Face; artículos | Cifras de referencia para las sondas de AVA [B4 §2.4, §2.5]. |

### ¿Cómo se une el ASR con la diarización?

La forma estándar es la de WhisperX, que es la que adopta DASR: Whisper transcribe, un modelo wav2vec 2.0 con CTC alinea cada palabra y la unión con la diarización se hace por intersección temporal [B3 §1.1–1.3].

La investigación reciente explora modelos conjuntos (DiCoW, Speech LLM, SpeakerLM) y post-procesado con LLM (DiarizationLM) [B3 §2]. Aun así, dos resultados indican que una cascada bien hecha no está en desventaja: el sistema de BUT en el reto MLC-SLM [B3 §2.7] y la evaluación de Amorim et al. (2026) [B1 §5.5].

### ¿Qué se puede decir de los atributos vocales?

Hay pocos resultados sólidos fuera de los modelos entrenados o afinados para cada atributo:

- **Sexo percibido:** es fácil; supera el 99 % con WavLM [B4 §2.5].
- **Edad:** es difícil, con errores medios de entre 5 y 11 años según el modelo y el dataset [B4 §2.3–2.5].
- **Acento:** depende de tener datos etiquetados de las variedades que interesan [B4 §3.1].

Los embeddings de hablante sí guardan información más allá de la identidad, pero también de canal y de contenido hablado [B4 §1.1]. Las capas intermedias de los modelos autosupervisados son mejores entradas que los embeddings especializados [B4 §1.2].

### ¿Cómo se evalúa y con qué datos?

- **Diarización:** DER sin collar y contando solapamiento, como lo reporta pyannote, con `pyannote.metrics` [B5 §1.1], más el JER [B5 §1.2].
- **ASR:** WER con `jiwer` [B3 §3.1].
- **Quién dijo qué:** cpWER o tcpWER con MeetEval [B3 §3.2, §3.3].
- **Datos libres más útiles:** AMI y VoxConverse [B5 §2.1, §2.3].
- **Datos en español:** RTVE (con licencia gratuita para investigación) [B5 §4.1], más grabaciones propias anotadas.

## Lo que dice el código de pyannote 3.1.1

Esto no es literatura, pero condiciona la reproducción. Se comprobó en el código de la etiqueta `3.1.1` y en el CHANGELOG oficial.

1. **Ventana y paso.** `segmentation_step = 0.1`, es decir, el paso es el 10 % de la duración de la ventana.
2. **Embeddings sin solapamiento.** `get_embeddings(exclude_overlap=True)` pone a cero los frames donde la suma de la segmentación binaria es 2 o más. Si la voz limpia que queda es demasiado corta para el modelo de embedding, usa la máscara completa.
3. **Clustering.** `AgglomerativeClustering` normaliza los embeddings y llama a `scipy.cluster.hierarchy.linkage(..., method="centroid", metric="euclidean")`. Corta con `fcluster(..., criterion="distance")` y reasigna los clústeres pequeños. Con pocos embeddings, `min_cluster_size` baja a `max(1, round(0.1 · N))`.
4. **Asignación por ventana.** `assign_embeddings` usa `scipy.optimize.linear_sum_assignment` (algoritmo húngaro).
5. **Conteo de hablantes.** `speaker_count` descarta el 10 % de cada borde de la ventana antes de promediar y redondea.
6. **Reconstrucción.** `reconstruct` toma, dentro de cada ventana, el máximo de las probabilidades de los locales asignados a un clúster, y `to_diarization` suma entre ventanas y elige los `Kf` mejores.
7. **Cambio en la 4.0.** Desde la versión 4.0.0 (septiembre de 2025) el clustering por defecto es `VBxClustering` con PLDA, así que la comparación de DASR debe hacerse contra la 3.1.
8. **Configuración oficial.** El `config.yaml` del modelo 3.1 en Hugging Face pide iniciar sesión; los valores δ ≈ 0.7046 y `min_cluster_size = 12` provienen de copias públicas de esa configuración.

## Ideas transferibles, consolidadas

1. **Validar tensor por tensor contra pyannote 3.1.1**, documentando cada etapa como hace el tutorial de DiariZen [B1 §4.3].
2. **Medir la calidad del clustering por sí misma** (pureza, cobertura) además del DER, porque predice el rendimiento final [B2 §3.8, B5 §1.1].
3. **Comparar AHC con centroides y VBx sobre los mismos embeddings locales**, que es justo el cambio que hizo pyannote 4.0 [B1 §1.6, B2 §3.5].
4. **Usar la diarización exclusiva en la fusión con el ASR** para eliminar la ambigüedad en el solapamiento [B1 §1.6, B3 §2.1].
5. **Reportar el DER por condición** (número de hablantes, ruido, duración), no solo el promedio [B1 §5.4, §5.5].
6. **Ajustar δ, `min_cluster_size` y los umbrales antes de afinar la segmentación** [B1 §1.1, §5.5].
7. **En AVA, entrenar la misma sonda sobre el centroide ResNet34 y sobre una capa intermedia de WavLM** y comparar UAR y MAE [B4 §1.2, §2.5].
8. **Particiones por hablante y evaluación por subgrupos** en AVA, reportando probabilidades [B4 §6.1].
9. **cpWER o tcpWER con MeetEval** para medir la salida conjunta de quién dijo qué [B3 §3.2, §3.3].
10. **DiarizationLM como extensión** sobre la salida de DASR, sin reentrenar nada [B3 §2.3].
11. **WavLM en la Fase 1 como extensión**, con la poda de DiariZen para que sea viable [B1 §4.1, §4.2].
12. **Streaming con diart** si se quiere una versión en tiempo real sin cambiar de arquitectura [B1 §1.5].

## Huecos: dónde puede estar la contribución

1. **No hay una comparación publicada entre AHC con centroides y VBx** sobre los mismos embeddings locales de pyannote.
2. **Faltan resultados de pyannote 3.1 en español** separados por idioma, y no hay un benchmark libre de diarización en español comparable a VoxConverse.
3. **Ninguna reproducción de pyannote 3.1 está documentada paso a paso** con tensores intermedios verificables.
4. **No se ha medido la sensibilidad de pyannote 3.1 al ruido y a la duración**, que es lo que más perjudica según Amorim et al. (2026).
5. **No hay resultados de atributos a partir de los centroides de un pipeline de diarización**, que es la entrada de AVA, ni del efecto de los errores de diarización sobre esos atributos.
6. **No existe un protocolo común para evaluar la cadena completa** diarización + ASR + atributos.

## Ruta de lectura sugerida

Pensada para leer con calma, de lo más cercano a lo más lejano. Todos los PDF de esta lista están en [`pdfs/`](pdfs/), salvo el de Amorim et al. (2026), que es de acceso abierto pero hay que bajarlo desde el navegador.

**Primero, el método que se reproduce.**
1. Bredin (2023), pyannote.audio 2.1 [B1 §1.1].
2. Bredin y Laurent (2021), segmentación end-to-end [B1 §1.3].
3. Plaquet y Bredin (2023), powerset [B1 §1.4].
4. SincNet y WeSpeaker, las piezas de las Fases 1 y 2 [B2 §1.3, §1.8].
5. Müllner (2011), clustering jerárquico y enlace de centroides [B2 §3.6].
6. VBx [B2 §3.5].

**Después, el contexto y la evaluación.**
7. DiariZen y su tutorial [B1 §4.1, §4.3].
8. Benchmarking Diarization Models (2025) y Amorim et al. (2026) [B1 §5.4, §5.5].
9. pyannote.metrics y DIHARD II [B5 §1.1, §1.2].

**La rama ASR.**
10. CTC, Whisper y WhisperX [B3 §1.1–1.3].
11. MeetEval [B3 §3.3].

**Atributos.**
12. Raj et al. (2019) y Chiu et al. (2025) [B4 §1.1, §1.2].
13. Burkhardt et al. (2023) y Yang et al. (2025) [B4 §2.4, §2.5].
14. CommonAccent [B4 §3.1] y Hutiri y Ding (2022) [B4 §6.1].

## Pendiente de verificar

- Venues que en arXiv solo constan como enviadas: MSDD (Interspeech 2022), DiariZen (ICASSP 2025), TitaNet (ICASSP 2022), CAM++ (Interspeech 2023), HuBERT (TASLP), Burkhardt et al. (ITG 2023), Yang et al. (CISS 2025), Pepino et al. (Interspeech 2021) y el taller de CHiME-6.
- El contenido de Kwasny y Hemmerling (2021) [B4 §2.2] y de Fathan y Alam (2024) [B2 §3.8]: el segundo se resumió a partir de una reseña.
- La ubicación del código de DiarizationLM y el nombre exacto del modelo de audEERING en Hugging Face.
- Los valores oficiales de δ y `min_cluster_size` de pyannote 3.1 (la configuración pide iniciar sesión).
- Todas las marcas ✓ de la matriz: falta la cita textual del PDF en la hoja *Evidencia*.

