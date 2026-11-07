# 3. Diarización end-to-end e híbrida

En lugar de encadenar etapas, una red neuronal recibe el audio y predice, **para cada frame**, qué hablantes están activos. Es una clasificación multi-etiqueta, así que el solapamiento sale de forma natural.

## 3.1 La línea EEND

| Modelo | Año | Aporte |
|--------|-----|--------|
| **EEND / SA-EEND** ([Fujita et al.](https://arxiv.org/abs/1909.06247)) | 2019 | Transformer que predice actividad por frame. Se entrena con **PIT** (*permutation-invariant training*) porque el orden de los hablantes es arbitrario. Número de hablantes fijo. |
| **EEND-EDA** ([Horiguchi et al.](https://arxiv.org/abs/2005.09921)) | 2020 | *Encoder-decoder attractors*: genera un "atractor" por hablante hasta decidir que no hay más. Número de hablantes variable. |
| **EEND-VC** ([Kinoshita et al.](https://arxiv.org/abs/2010.13366)) | 2020 | EEND en bloques cortos + **clustering de vectores** entre bloques. Es el origen del enfoque híbrido actual. |
| **TS-VAD** ([Medennikov et al.](https://arxiv.org/abs/2005.07272)) | 2020 | *Target-speaker VAD*: dada una diarización inicial, refina la actividad de cada hablante con su embedding. Ganó CHiME-6; sus variantes (NSD-MA-MSE, etc.) dominaron retos de reuniones. |
| **EEND-TA** ([Broughton & Samarakoon, Interspeech 2025](https://www.isca-archive.org/interspeech_2025/broughton25_interspeech.html)) | 2025 | Muestra que EEND tiene más capacidad de la explorada: ~14.5% de DER en DIHARD III. |
| **O-EENC-SD** ([2025](https://arxiv.org/abs/2512.15229)) | 2025 | EEND-EDA en línea (*online*) con un mecanismo de "costura" con RNN entre bloques. |

**Problema de EEND puro:** necesita muchísimos datos anotados (se entrena con mezclas simuladas y luego se adapta) y escala mal con audios largos y muchos hablantes.

## 3.2 Pérdida powerset

[Plaquet & Bredin, 2023](https://arxiv.org/abs/2310.13025). En vez de una salida sigmoide por hablante (multi-etiqueta) con un umbral, se predice **una clase por cada combinación** de hablantes activos ("nadie", "A", "B", "A+B"…). Con hasta 3 hablantes por ventana y 2 simultáneos salen 7 clases. Ventajas: no hay umbral que ajustar y el solapamiento se modela mejor. Lo adoptaron pyannote 3.x y DiariZen.

## 3.3 DiariZen (BUT, 2024–2026): el mejor sistema abierto

- Papers: [Han et al., 2024 — *Leveraging Self-Supervised Learning for Speaker Diarization*](https://arxiv.org/abs/2409.09408) y [Han et al., 2025 — *Efficient and Robust Speaker Diarization via Structured Pruning of Self-Supervised Models*](https://arxiv.org/abs/2506.18623).
- Tutorial: [Raghav, abril 2026 — *DiariZen Explained*](https://arxiv.org/abs/2604.21507), con un repo de scripts por bloque y un notebook completo. **Buen punto de partida práctico para el proyecto.**
- Arquitectura: **WavLM-Large podado** (pruning estructurado) → **Conformer** → salida **powerset** en ventanas locales → embeddings → **VBx** para la identidad global. Es la idea de pyannote con un codificador autosupervisado mucho más fuerte.
- Resultados: en el benchmark de ETH 2025 es el mejor sistema abierto (13.3% de DER medio). Es el mejor en VoxConverse (5.2%) y con 5 o más hablantes (7.1%). BUT lo combinó con DiCoW y quedó 2.º en el reto MLC-SLM 2025 ([BUT System](https://arxiv.org/abs/2506.13414)).
- *Revisar la licencia de cada checkpoint: algunos modelos entrenados con datos académicos son de uso no comercial.*

## 3.4 Sortformer y la línea de NVIDIA

| Modelo | Fecha | Aporte |
|--------|-------|--------|
| **Sortformer** ([Park et al.](https://arxiv.org/abs/2409.06656)) | sep. 2024 | Reemplaza PIT por **ordenar los hablantes por orden de llegada** (*arrival-time ordering*, "Sort Loss"). Así la salida se puede pegar directamente a un ASR. Máximo 4 hablantes. |
| **Streaming Sortformer** ([arXiv 2507.18446](https://arxiv.org/abs/2507.18446)) | jul. 2025 | Versión en tiempo real con una **caché de hablantes** ordenada por llegada (AOSC) y una cola FIFO para mantener identidades entre bloques. |
| **Nemotron 3 Diarization** ([ficha en Hugging Face](https://huggingface.co/nvidia/Nemotron-3-Diarization)) | sep. 2026 | ~100M parámetros, **hasta 8 hablantes**, modo offline y streaming con latencias configurables (30.4 s, 1.04 s, 0.64 s, 0.32 s). Pesos abiertos (licencia OpenMDW-1.1), se usa con NeMo. |

Según el benchmark de ETH 2025, Sortformer v2-streaming es el mejor en AliMeeting (7.0%), pero **todos los Sortformer empeoran al crecer el número de hablantes** por el límite de 4. Nemotron 3 ataca justo eso. Baseten reporta 9.8% de DER en AISHELL-4 en baja latencia frente a 27.2% de Streaming Sortformer v2.1. *Son cifras del fabricante y sus socios, no una evaluación independiente.*

## 3.5 ¿Quién gana hoy? (a noviembre de 2026)

No hay un ganador único: **depende del tipo de audio**.

| Fuente | Resultado |
|--------|-----------|
| [Benchmarking Diarization Models](https://arxiv.org/abs/2509.26177) (ETH Zurich, sep. 2025; 196.6 h en 5 idiomas, **incluye español**) | 1.º pyannoteAI (comercial) 11.2%; mejor abierto DiariZen 13.3%; Sortformer v2-streaming muy fuerte con pocos hablantes |
| Voice Arena Diarization-Bench (leaderboard, 2026) | Reportan 1.º Nemotron 3 (14.7%), 2.º DiariZen; *community-1* 6.º (30.6%) en su conjunto con mucho solapamiento. *No lo pude verificar contra el leaderboard; sus condiciones parecen distintas a las del benchmark de ETH.* |

**Conclusión práctica:** para un proyecto académico con audios variados, las opciones abiertas más sólidas son **DiariZen** (precisión offline) y **pyannote community-1** (facilidad de uso y documentación). **Nemotron 3** es la opción si se quiere streaming o si hay más de 4 hablantes.
