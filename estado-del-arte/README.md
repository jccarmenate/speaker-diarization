# Estado del arte: identificación de hablantes y diarización

Revisión actualizada a noviembre de 2026 para el proyecto final de Machine Learning.

**Objetivo del proyecto:** dado un audio o video, saber **quién** habló, **qué** dijo, **cuándo** lo dijo y **dónde** (en qué parte del archivo y, en video, cuál de las personas en pantalla).

## Índice por subtemas

| # | Archivo | Qué cubre |
|---|---------|-----------|
| 1 | [Problema y tareas](01-problema-y-tareas.md) | Cómo se descompone "quién dijo qué, cuándo y dónde" en tareas de ML |
| 2 | [Pipeline modular](02-pipeline-modular.md) | VAD, segmentación, embeddings y clustering (pyannote, VBx) |
| 3 | [Diarización end-to-end](03-end-to-end.md) | EEND, EEND-VC, TS-VAD, powerset, Sortformer, DiariZen, Nemotron 3 |
| 4 | [Embeddings e identificación](04-embeddings-e-identificacion.md) | x-vectors, ECAPA-TDNN, ResNet, CAM++, WavLM; identificar a personas conocidas |
| 5 | [Diarización + transcripción](05-diarizacion-y-transcripcion.md) | Whisper/WhisperX, DiCoW, DiarizationLM, Speech LLMs |
| 6 | [Audiovisual y localización](06-audiovisual-y-localizacion.md) | Detección de hablante activo, diarización audiovisual, el "dónde" |
| 7 | [Datasets y métricas](07-datasets-y-metricas.md) | DER, JER, WDER, cpWER/tcpWER, EER; corpus, incluido español |
| 8 | [Herramientas](08-herramientas.md) | Toolkits y modelos preentrenados disponibles |
| 9 | [Brechas y dirección del proyecto](09-brechas-y-direccion.md) | Problemas abiertos y una propuesta de pipeline para el proyecto |
| — | [Referencias](referencias.md) | Bibliografía completa con enlaces |

## Resumen en cinco puntos

1. **La diarización no está resuelta.** En el benchmark independiente más reciente (ETH Zurich, 2025), el mejor sistema comercial queda en 11.2% de DER y el mejor abierto (DiariZen) en 13.3%. La mayoría de los errores vienen de habla no detectada y de confundir hablantes cuando hay muchos.
2. **El estado del arte es híbrido.** Los mejores sistemas abiertos combinan un segmentador neuronal *end-to-end* en ventanas cortas (WavLM + Conformer con pérdida *powerset*) con *clustering* global (VBx o aglomerativo). Esa es la línea de pyannote y de DiariZen.
3. **Lo nuevo es streaming y más hablantes.** NVIDIA pasó de Sortformer (4 hablantes) a Streaming Sortformer y en septiembre de 2026 a Nemotron 3 Diarization (8 hablantes, latencia desde 0.32 s, pesos abiertos).
4. **"Quién dijo qué" se está unificando.** Cada vez más trabajos (2025–2026) hacen diarización y ASR juntos: Whisper condicionado por diarización (DiCoW) o *Speech LLMs* que generan texto con etiquetas de hablante (SpeakerLM, el paper 2507.02927).
5. **El video ayuda menos de lo esperado en reuniones.** En el reto MISP 2025, los ganadores reportan que la información visual casi no mejora a un buen sistema solo-audio. Para el "dónde" en video sigue siendo necesaria la detección de hablante activo.

## Revisión de los links que pasaste

| Link | ¿Sirve? | Qué es |
|------|---------|--------|
| [arxiv.org/abs/2507.02927](https://arxiv.org/abs/2507.02927) | **Sí** | *A Unified Speech LLM for Diarization and Speech Recognition in Multilingual Conversations* (Saengthong et al., junio 2025). Un solo modelo hace diarización + ASR, para la Task II del reto MLC-SLM. Mejora 54.87% relativo en tcpWER/tcpCER sobre el baseline y queda 8.º. Sirve como referencia de la línea "Speech LLM" (subtema 5). |
| [arxiv.org/abs/2507.02927?utm_source=chatgpt.com](https://arxiv.org/abs/2507.02927?utm_source=chatgpt.com) | Duplicado | Es el mismo paper. El `?utm_source=chatgpt.com` es solo un parámetro de rastreo; conviene citarlo sin él. |
| [isca-archive.org/…/bredin23_interspeech](https://www.isca-archive.org/interspeech_2023/bredin23_interspeech.html) | **Sí, es clave** | *pyannote.audio 2.1 speaker diarization pipeline: principle, benchmark, and recipe* (Bredin, Interspeech 2023). Describe el pipeline de tres etapas (segmentación local → embeddings → clustering aglomerativo) que es la base de pyannote y de DiariZen. Incluye una receta para adaptarlo a datos propios. Fue 1.º en Ego4D 2022 y Albayzin 2022. |

**Ojo:** el paper de pyannote 2.1 ya no describe la versión actual. Hoy se usa pyannote 3.1 / *community-1* (pyannote.audio 4.0), que cambia el segmentador a *powerset* y los embeddings a ResNet de WeSpeaker. El principio es el mismo, así que sigue sirviendo para explicar el método.
