# Frente 1: sistemas de diarización de hablantes

Revisión bibliográfica para DASR: qué sistemas de diarización existen, dónde queda pyannote 3.1 entre ellos y cuáles son las alternativas reales. Fecha de la búsqueda: 12 de noviembre de 2026. Autores, año, título y enlace de cada referencia se comprobaron contra arXiv o Crossref. Las venues que en arXiv solo constan como "enviado" o que no se pudieron confirmar van marcadas "(no verificado)". El resumen de cada trabajo se basa en su abstract, salvo cuando se indica otra cosa.

Leyenda de etiquetas: [solapamiento] si el sistema modela habla simultánea, [nº hablantes] cuántos admite, [tiempo real] si funciona en streaming, [código], [relevancia] para DASR.

Los PDF de este banco están en [`pdfs/1-diarizacion/`](pdfs/1-diarizacion/); el [índice](pdfs/INDICE.md) lista los que hay que bajar a mano.

## 1. pyannote: el enfoque local + global que reproduce DASR

### 1.1 pyannote.audio 2.1
- Referencia: Hervé Bredin (2023). "pyannote.audio 2.1 speaker diarization pipeline: principle, benchmark, and recipe". Interspeech 2023, pp. 1983–1987. DOI: https://doi.org/10.21437/interspeech.2023-105.
- Qué hace: Describe el pipeline de diarización de pyannote.audio 2.1 en tres etapas: segmentación neuronal en ventanas deslizantes cortas que detecta hasta tres hablantes locales y su solapamiento, un embedding por hablante local y clustering aglomerativo global. Publica un benchmark en varios corpus y una receta para adaptar el pipeline a datos propios: afinar el modelo de segmentación y optimizar los hiperparámetros. Fue 1.º en Ego4D 2022 y Albayzin 2022 y 6.º en VoxSRC 2022.
- Relevancia para DASR: Es el artículo central del proyecto. DASR reproduce este pipeline (en su versión 3.1) fase por fase, y la receta de adaptación es la base del fine-tuning propuesto.
- Etiquetas: [solapamiento: sí] [nº hablantes: libre] [tiempo real: no] [código: https://github.com/pyannote/pyannote-audio] [relevancia: alta]
- PDF: [1.01-Bredin-2023-pyannote-audio-2-1-speaker-diarization-pipeline-principle.pdf](pdfs/1-diarizacion/1.01-Bredin-2023-pyannote-audio-2-1-speaker-diarization-pipeline-principle.pdf)

### 1.2 pyannote.audio (bloques neuronales)
- Referencia: Hervé Bredin, Ruiqing Yin, Juan Manuel Coria, Gregory Gelly, Pavel Korshunov, Marvin Lavechin, Diego Fustes, Hadrien Titeux et al. (2019). "pyannote.audio: neural building blocks for speaker diarization". ICASSP 2020. arXiv: https://arxiv.org/abs/1911.01255.
- Qué hace: Presenta el toolkit como un conjunto de bloques neuronales entrenables de punta a punta (detección de voz, de cambio de hablante y de solapamiento, embeddings y resegmentación) que se pueden optimizar conjuntamente respecto al objetivo final.
- Relevancia para DASR: Explica el diseño modular del que parte DASR y es la fuente de la figura de módulos del PPT (diapositiva 8).
- Etiquetas: [solapamiento: sí, con detector propio] [nº hablantes: libre] [tiempo real: no] [código: https://github.com/pyannote/pyannote-audio] [relevancia: media]
- PDF: [1.02-Bredin-2019-pyannote-audio-neural-building-blocks-for-speaker-diarization.pdf](pdfs/1-diarizacion/1.02-Bredin-2019-pyannote-audio-neural-building-blocks-for-speaker-diarization.pdf)

### 1.3 Segmentación end-to-end con resegmentación
- Referencia: Hervé Bredin, Antoine Laurent (2021). "End-to-end speaker segmentation for overlap-aware resegmentation". Interspeech 2021. arXiv: https://arxiv.org/abs/2104.04045.
- Qué hace: Entrena una sola red (SincNet + LSTM) que, en ventanas cortas, predice la actividad de hasta K hablantes locales con un objetivo invariante a permutaciones. Esa red sustituye a la detección de voz, de cambio y de solapamiento, y sirve para resegmentar con conciencia de solapamiento la salida de cualquier sistema de diarización.
- Relevancia para DASR: Es el origen de la Fase 1 de DASR: los hablantes locales por ventana y el problema de permutación entre ventanas vienen de aquí.
- Etiquetas: [solapamiento: sí] [nº hablantes: hasta K por ventana] [tiempo real: no] [código: https://github.com/pyannote/pyannote-audio] [relevancia: alta]
- PDF: [1.03-Bredin-2021-End-to-end-speaker-segmentation-for-overlap-aware.pdf](pdfs/1-diarizacion/1.03-Bredin-2021-End-to-end-speaker-segmentation-for-overlap-aware.pdf)

### 1.4 Pérdida powerset
- Referencia: Alexis Plaquet, Hervé Bredin (2023). "Powerset multi-class cross entropy loss for neural speaker diarization". Interspeech 2023, pp. 3222–3226. arXiv: https://arxiv.org/abs/2310.13025. DOI: https://doi.org/10.21437/Interspeech.2023-205 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Sustituye la salida multietiqueta (una sigmoide por hablante con umbral) por una clasificación multiclase sobre el conjunto potencia de hablantes: con 3 hablantes locales y como máximo 2 simultáneos salen 7 clases. Elimina el umbral de detección y mejora el rendimiento en solapamiento. Es el modelo `pyannote/segmentation-3.0` que usa pyannote 3.x.
- Relevancia para DASR: Define la salida de la red de la Fase 1 de DASR y la conversión de powerset a multietiqueta (`utils/powerset.py`).
- Etiquetas: [solapamiento: sí] [nº hablantes: 3 por ventana, 2 a la vez] [tiempo real: no] [código: https://github.com/pyannote/pyannote-audio] [relevancia: alta]
- PDF: [1.04-Plaquet-2023-Powerset-multi-class-cross-entropy-loss-for-neural.pdf](pdfs/1-diarizacion/1.04-Plaquet-2023-Powerset-multi-class-cross-entropy-loss-for-neural.pdf)

### 1.5 diart (diarización en línea)
- Referencia: Juan M. Coria, Hervé Bredin, Sahar Ghannay, Sophie Rosset (2021). "Overlap-aware low-latency online speaker diarization based on end-to-end local segmentation". ASRU 2021. arXiv: https://arxiv.org/abs/2109.06483.
- Qué hace: Adapta la segmentación local de pyannote a diarización en línea con baja latencia: procesa el audio en ventanas deslizantes, extrae embeddings por hablante local y los asigna de forma incremental a centroides globales, teniendo en cuenta el solapamiento.
- Relevancia para DASR: Es la versión en tiempo real del mismo enfoque local + global. Sirve si el proyecto quiere una extensión en streaming sin cambiar de arquitectura.
- Etiquetas: [solapamiento: sí] [nº hablantes: libre] [tiempo real: sí] [código: https://github.com/juanmc2005/diart] [relevancia: media]
- PDF: [1.05-Coria-2021-Overlap-aware-low-latency-online-speaker-diarization-based.pdf](pdfs/1-diarizacion/1.05-Coria-2021-Overlap-aware-low-latency-online-speaker-diarization-based.pdf)

### 1.6 pyannote community-1 (pyannote.audio 4.0)
- Referencia: pyannoteAI (2025). "pyannote/speaker-diarization-community-1 y pyannote.audio 4.0". ficha del modelo y registro de cambios. https://huggingface.co/pyannote/speaker-diarization-community-1. CHANGELOG 4.0.0: https://github.com/pyannote/pyannote-audio/blob/develop/CHANGELOG.md.
- Qué hace: Pipeline abierto actual de pyannote (septiembre de 2025, licencia CC-BY-4.0). Según el CHANGELOG de la versión 4.0.0, sustituye el clustering aglomerativo por VBx (sugerido por investigadores de BUT) y añade una diarización exclusiva, sin solapamiento, para conciliar con las marcas de tiempo del ASR. La ficha reporta menor DER que 3.1 en todos sus benchmarks.
- Relevancia para DASR: Es lo que vino después de la versión que reproduce DASR, y fija dos extensiones naturales: VBx y la diarización exclusiva para la fusión con el ASR.
- Etiquetas: [solapamiento: sí, más salida exclusiva] [nº hablantes: libre] [tiempo real: no] [código: https://github.com/pyannote/pyannote-audio; modelo con acceso restringido en Hugging Face] [relevancia: alta]
- PDF: no aplica (página web, sin PDF)

## 2. Diarización end-to-end (EEND y derivados)

### 2.1 EEND
- Referencia: Yusuke Fujita, Naoyuki Kanda, Shota Horiguchi, Kenji Nagamatsu, Shinji Watanabe (2019). "End-to-End Neural Speaker Diarization with Permutation-Free Objectives". Interspeech 2019. arXiv: https://arxiv.org/abs/1909.05952.
- Qué hace: Primera formulación de la diarización como clasificación multietiqueta por frame con una BLSTM, entrenada con objetivos libres de permutación (PIT). Maneja el solapamiento de forma natural, pero el número de hablantes es fijo.
- Relevancia para DASR: Origen del entrenamiento con PIT, que es el mismo problema de permutación que DASR resuelve con clustering.
- Etiquetas: [solapamiento: sí] [nº hablantes: fijo] [tiempo real: no] [código: https://github.com/hitachi-speech/EEND] [relevancia: media]
- PDF: [2.01-Fujita-2019-End-to-End-Neural-Speaker-Diarization-with-Permutation.pdf](pdfs/1-diarizacion/2.01-Fujita-2019-End-to-End-Neural-Speaker-Diarization-with-Permutation.pdf)

### 2.2 SA-EEND
- Referencia: Yusuke Fujita, Naoyuki Kanda, Shota Horiguchi, Yawen Xue, Kenji Nagamatsu, Shinji Watanabe (2019). "End-to-End Neural Speaker Diarization with Self-attention". ASRU 2019. arXiv: https://arxiv.org/abs/1909.06247.
- Qué hace: Sustituye la BLSTM de EEND por self-attention (Transformer) y supera a los sistemas de clustering con x-vectors en conversaciones de dos hablantes.
- Relevancia para DASR: Arquitectura base de casi toda la línea EEND posterior.
- Etiquetas: [solapamiento: sí] [nº hablantes: fijo] [tiempo real: no] [código: https://github.com/hitachi-speech/EEND] [relevancia: baja]
- PDF: [2.02-Fujita-2019-End-to-End-Neural-Speaker-Diarization-with-Self.pdf](pdfs/1-diarizacion/2.02-Fujita-2019-End-to-End-Neural-Speaker-Diarization-with-Self.pdf)

### 2.3 EEND-EDA
- Referencia: Shota Horiguchi, Yusuke Fujita, Shinji Watanabe, Yawen Xue, Kenji Nagamatsu (2020). "End-to-End Speaker Diarization for an Unknown Number of Speakers with Encoder-Decoder Based Attractors". Interspeech 2020. arXiv: https://arxiv.org/abs/2005.09921.
- Qué hace: Añade attractors calculados con un codificador-decodificador LSTM: genera un attractor por hablante y decide cuándo parar, lo que admite un número variable de hablantes.
- Relevancia para DASR: Alternativa end-to-end al conteo de hablantes con el umbral δ de la Fase 3.
- Etiquetas: [solapamiento: sí] [nº hablantes: variable, hasta el máximo visto en entrenamiento] [tiempo real: no] [código: https://github.com/hitachi-speech/EEND] [relevancia: media]
- PDF: [2.03-Horiguchi-2020-End-to-End-Speaker-Diarization-for-an-Unknown.pdf](pdfs/1-diarizacion/2.03-Horiguchi-2020-End-to-End-Speaker-Diarization-for-an-Unknown.pdf)

### 2.4 EEND-VC
- Referencia: Keisuke Kinoshita, Marc Delcroix, Naohiro Tawara (2020). "Integrating end-to-end neural and clustering-based diarization: Getting the best of both worlds". ICASSP 2021. arXiv: https://arxiv.org/abs/2010.13366.
- Qué hace: Aplica EEND a bloques cortos y extrae un vector por cada hablante del bloque; después agrupa esos vectores entre bloques con clustering restringido, para que dos hablantes del mismo bloque no queden en el mismo grupo.
- Relevancia para DASR: Es la misma idea local + global de pyannote, formulada en paralelo. Su restricción por bloque es el análogo de la asignación húngara por ventana de la Fase 3.
- Etiquetas: [solapamiento: sí] [nº hablantes: libre] [tiempo real: no] [código: https://github.com/nttcslab-sp/EEND-vector-clustering] [relevancia: alta]
- PDF: [2.04-Kinoshita-2020-Integrating-end-to-end-neural-and-clustering-based.pdf](pdfs/1-diarizacion/2.04-Kinoshita-2020-Integrating-end-to-end-neural-and-clustering-based.pdf)

### 2.5 TS-VAD
- Referencia: Ivan Medennikov, Maxim Korenevsky, Tatiana Prisyach, Yuri Khokhlov, Mariya Korenevskaya, Ivan Sorokin, Tatiana Timofeeva, Anton Mitrofanov et al. (2020). "Target-Speaker Voice Activity Detection: a Novel Approach for Multi-Speaker Diarization in a Dinner Party Scenario". Interspeech 2020. arXiv: https://arxiv.org/abs/2005.07272. DOI: https://doi.org/10.21437/Interspeech.2020-1602 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Partiendo de una diarización inicial, toma un embedding (i-vector) por hablante y una red estima, para cada hablante objetivo, si habla en cada frame. Fue clave en el sistema ganador de CHiME-6.
- Relevancia para DASR: Muestra una mejora posterior al clustering: re-estimar la actividad de cada hablante global con su embedding.
- Etiquetas: [solapamiento: sí] [nº hablantes: fijo en el modelo original] [tiempo real: no] [código: no localizado] [relevancia: media]
- PDF: [2.05-Medennikov-2020-Target-Speaker-Voice-Activity-Detection-a-Novel-Approach.pdf](pdfs/1-diarizacion/2.05-Medennikov-2020-Target-Speaker-Voice-Activity-Detection-a-Novel-Approach.pdf)

### 2.6 NSD-MS2S
- Referencia: Gaobin Yang, Maokui He, Shutong Niu, Ruoyu Wang, Yanyan Yue, Shuangqing Qian, Shilong Wu, Jun Du et al. (2023). "Neural Speaker Diarization Using Memory-Aware Multi-Speaker Embedding with Sequence-to-Sequence Architecture". ICASSP 2024. arXiv: https://arxiv.org/abs/2309.09180.
- Qué hace: Diarización neuronal secuencia a secuencia con embeddings multi-hablante de memoria (MA-MSE). Obtiene un DER macro de 15.9 % en la evaluación de CHiME-7 y fue la técnica clave del mejor sistema de la pista principal de CHiME-7 DASR.
- Relevancia para DASR: Referencia del estado del arte en reuniones difíciles; queda fuera de lo que se puede reproducir en el curso.
- Etiquetas: [solapamiento: sí] [nº hablantes: máximo configurable] [tiempo real: no] [código: no localizado] [relevancia: baja]
- PDF: [2.06-Yang-2023-Neural-Speaker-Diarization-Using-Memory-Aware-Multi-Speaker.pdf](pdfs/1-diarizacion/2.06-Yang-2023-Neural-Speaker-Diarization-Using-Memory-Aware-Multi-Speaker.pdf)

### 2.7 LS-EEND
- Referencia: Di Liang, Xiaofei Li (2024). "LS-EEND: Long-Form Streaming End-to-End Neural Diarization with Online Attractor Extraction". IEEE/ACM TASLP. arXiv: https://arxiv.org/abs/2410.06670.
- Qué hace: EEND en streaming frame a frame con un codificador causal y un decodificador de attractors en línea; un mecanismo de retención da complejidad lineal en audio largo.
- Relevancia para DASR: Alternativa si se quisiera diarización en tiempo real con un solo modelo.
- Etiquetas: [solapamiento: sí] [nº hablantes: variable] [tiempo real: sí] [código: no localizado] [relevancia: baja]
- PDF: [2.07-Liang-2024-LS-EEND-Long-Form-Streaming-End-to-End.pdf](pdfs/1-diarizacion/2.07-Liang-2024-LS-EEND-Long-Form-Streaming-End-to-End.pdf)

### 2.8 O-EENC-SD
- Referencia: Elio Gruttadauria, Mathieu Fontaine, Jonathan Le Roux, Slim Essid (2025). "O-EENC-SD: Efficient Online End-to-End Neural Clustering for Speaker Diarization". ICASSP 2026. arXiv: https://arxiv.org/abs/2512.15229.
- Qué hace: Diarización en línea basada en EEND-EDA con un mecanismo de "costura" con RNN que mantiene las identidades entre bloques.
- Relevancia para DASR: Aborda desde EEND el mismo problema de identidad entre bloques que DASR resuelve con clustering.
- Etiquetas: [solapamiento: sí] [nº hablantes: variable] [tiempo real: sí] [código: no localizado] [relevancia: baja]
- PDF: [2.08-Gruttadauria-2025-O-EENC-SD-Efficient-Online-End-to-End.pdf](pdfs/1-diarizacion/2.08-Gruttadauria-2025-O-EENC-SD-Efficient-Online-End-to-End.pdf)

### 2.9 EEND-TA a mayor escala
- Referencia: Samuel J. Broughton, Lahiru Samarakoon (2025). "Pushing the Limits of End-to-End Diarization". Interspeech 2025. DOI: https://doi.org/10.21437/interspeech.2025-1150.
- Qué hace: Revisa la diarización end-to-end con attractors basados en Transformer y muestra que, entrenada a mayor escala, supera a muchas soluciones de diarización manteniendo una inferencia eficiente.
- Relevancia para DASR: Evidencia de que la línea end-to-end sigue siendo competitiva; contraste con la elección modular de DASR.
- Etiquetas: [solapamiento: sí] [nº hablantes: variable] [tiempo real: no] [código: no localizado] [relevancia: baja]
- PDF: [2.09-Broughton-2025-Pushing-the-Limits-of-End-to-End-Diarization.pdf](pdfs/1-diarizacion/2.09-Broughton-2025-Pushing-the-Limits-of-End-to-End-Diarization.pdf)

## 3. La línea de NVIDIA (NeMo)

### 3.1 MSDD
- Referencia: Tae Jin Park, Nithin Rao Koluguri, Jagadeesh Balam, Boris Ginsburg (2022). "Multi-scale Speaker Diarization with Dynamic Scale Weighting". Interspeech 2022 (no verificado). arXiv: https://arxiv.org/abs/2203.15974.
- Qué hace: Diarización multiescala: un clustering inicial en varias escalas estima el número de hablantes y un decodificador con convoluciones 1D decide, en cada instante, cuánto pesa cada escala temporal.
- Relevancia para DASR: Otra respuesta al dilema de DASR entre identidad fiable (ventanas largas) y resolución temporal (ventanas cortas).
- Etiquetas: [solapamiento: sí] [nº hablantes: libre] [tiempo real: no] [código: https://github.com/NVIDIA-NeMo/Speech] [relevancia: media]
- PDF: [3.01-Park-2022-Multi-scale-Speaker-Diarization-with-Dynamic-Scale-Weighting.pdf](pdfs/1-diarizacion/3.01-Park-2022-Multi-scale-Speaker-Diarization-with-Dynamic-Scale-Weighting.pdf)

### 3.2 Sortformer
- Referencia: Taejin Park, Ivan Medennikov, Kunal Dhawan, Weiqing Wang, He Huang, Nithin Rao Koluguri, Krishna C. Puvvada, Jagadeesh Balam et al. (2025). "Sortformer: A Novel Approach for Permutation-Resolved Speaker Supervision in Speech-to-Text Systems". ICML 2025. arXiv: https://arxiv.org/abs/2409.06656.
- Qué hace: Red end-to-end que resuelve la permutación ordenando a los hablantes por orden de llegada (Sort Loss) en lugar de PIT, lo que permite conectar la diarización directamente con un ASR multi-hablante. Admite hasta 4 hablantes.
- Relevancia para DASR: Contraste directo: resuelve dentro de la red lo que DASR resuelve con un algoritmo visible.
- Etiquetas: [solapamiento: sí] [nº hablantes: hasta 4] [tiempo real: no] [código: https://github.com/NVIDIA-NeMo/Speech] [relevancia: media]
- PDF: [3.02-Park-2025-Sortformer-A-Novel-Approach-for-Permutation-Resolved-Speaker.pdf](pdfs/1-diarizacion/3.02-Park-2025-Sortformer-A-Novel-Approach-for-Permutation-Resolved-Speaker.pdf)

### 3.3 Streaming Sortformer
- Referencia: Ivan Medennikov, Taejin Park, Weiqing Wang, He Huang, Kunal Dhawan, Jinhan Wang, Jagadeesh Balam, Boris Ginsburg (2025). "Streaming Sortformer: Speaker Cache-Based Online Speaker Diarization with Arrival-Time Ordering". Interspeech 2025. arXiv: https://arxiv.org/abs/2507.18446.
- Qué hace: Versión en línea de Sortformer con una caché de hablantes ordenada por llegada (AOSC) y una cola FIFO que conserva las identidades entre bloques.
- Relevancia para DASR: Referencia de diarización en tiempo real; en el benchmark de 2025 [B1 §5.4] es muy fuerte con pocos hablantes.
- Etiquetas: [solapamiento: sí] [nº hablantes: hasta 4] [tiempo real: sí] [código: https://github.com/NVIDIA-NeMo/Speech] [relevancia: media]
- PDF: [3.03-Medennikov-2025-Streaming-Sortformer-Speaker-Cache-Based-Online-Speaker-Diarization.pdf](pdfs/1-diarizacion/3.03-Medennikov-2025-Streaming-Sortformer-Speaker-Cache-Based-Online-Speaker-Diarization.pdf)

### 3.4 Nemotron 3 Diarization
- Referencia: NVIDIA (2026). "Nemotron 3 Diarization (ficha del modelo)". Hugging Face. https://huggingface.co/nvidia/Nemotron-3-Diarization.
- Qué hace: Modelo abierto de unos 100 M de parámetros (septiembre de 2026) que sigue a Sortformer y Streaming Sortformer: admite hasta 8 hablantes y funciona offline o en streaming, con latencias de 0.32 a 30.4 s. Licencia OpenMDW-1.1. Las cifras de DER publicadas vienen del fabricante y de sus socios.
- Relevancia para DASR: El sistema más reciente de la línea end-to-end; buena comparación externa si hay GPU.
- Etiquetas: [solapamiento: sí] [nº hablantes: hasta 8] [tiempo real: sí] [código: https://github.com/NVIDIA-NeMo/Speech; pesos en Hugging Face] [relevancia: media]
- PDF: no aplica (página web, sin PDF)

## 4. DiariZen: el mismo enfoque con modelos autosupervisados

### 4.1 DiariZen
- Referencia: Jiangyu Han, Federico Landini, Johan Rohdin, Anna Silnova, Mireia Diez, Lukas Burget (2024). "Leveraging Self-Supervised Learning for Speaker Diarization". ICASSP 2025 (no verificado). arXiv: https://arxiv.org/abs/2409.09408.
- Qué hace: Sustituye la red de segmentación de pyannote por WavLM afinado con un Conformer y salida powerset, y mantiene el enfoque local + global con clustering VBx. Mejora a pyannote en varios corpus.
- Relevancia para DASR: Es la evolución directa de lo que reproduce DASR: cambia la pieza de la Fase 1 y deja la arquitectura.
- Etiquetas: [solapamiento: sí] [nº hablantes: libre] [tiempo real: no] [código: https://github.com/BUTSpeechFIT/DiariZen] [relevancia: alta]
- PDF: [4.01-Han-2024-Leveraging-Self-Supervised-Learning-for-Speaker-Diarization.pdf](pdfs/1-diarizacion/4.01-Han-2024-Leveraging-Self-Supervised-Learning-for-Speaker-Diarization.pdf)

### 4.2 Poda estructurada de DiariZen
- Referencia: Jiangyu Han, Petr Pálka, Marc Delcroix, Federico Landini, Johan Rohdin, Jan Cernocký, Lukáš Burget (2026). "Efficient and Robust Speaker Diarization via Structured Pruning of Self-Supervised Models". IEEE/ACM TASLP, vol. 34. arXiv: https://arxiv.org/abs/2506.18623. DOI: https://doi.org/10.1109/TASLPRO.2026.3675801 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Comprime el modelo autosupervisado de DiariZen con poda estructurada guiada por destilación: hasta un 80 % menos de parámetros e inferencia 4 veces más rápida sin perder precisión, en ocho datasets públicos.
- Relevancia para DASR: Muestra cómo hacer viable un modelo autosupervisado grande, que es la principal barrera para usar DiariZen en el curso.
- Etiquetas: [solapamiento: sí] [nº hablantes: libre] [tiempo real: no] [código: https://github.com/BUTSpeechFIT/DiariZen] [relevancia: media]
- PDF: [4.02-Han-2026-Efficient-and-Robust-Speaker-Diarization-via-Structured-Pruning.pdf](pdfs/1-diarizacion/4.02-Han-2026-Efficient-and-Robust-Speaker-Diarization-via-Structured-Pruning.pdf)

### 4.3 DiariZen Explained (tutorial)
- Referencia: Nikhil Raghav (2026). "DiariZen Explained: A Tutorial for the Open Source State-of-the-Art Speaker Diarization Pipeline". arXiv. arXiv: https://arxiv.org/abs/2604.21507.
- Qué hace: Tutorial que descompone DiariZen en siete etapas, con un script por bloque y un notebook que ejecuta el pipeline completo.
- Relevancia para DASR: Modelo de cómo documentar una reproducción paso a paso, que es justo lo que hace DASR con pyannote; su repositorio sirve de guía para comparar.
- Etiquetas: [solapamiento: sí] [nº hablantes: libre] [tiempo real: no] [código: https://github.com/nikhilraghav29/diarizen-tutorial] [relevancia: alta]
- PDF: [4.03-Raghav-2026-DiariZen-Explained-A-Tutorial-for-the-Open-Source.pdf](pdfs/1-diarizacion/4.03-Raghav-2026-DiariZen-Explained-A-Tutorial-for-the-Open-Source.pdf)

## 5. Revisiones, benchmarks y limitaciones

### 5.1 Revisión de Anguera et al.
- Referencia: Xavier Anguera Miro, S. Bozonnet, N. Evans, C. Fredouille, G. Friedland, O. Vinyals (2012). "Speaker Diarization: A Review of Recent Research". IEEE TASLP 20(2). DOI: https://doi.org/10.1109/tasl.2011.2125954.
- Qué hace: Revisión clásica de la diarización anterior al deep learning: detección de voz, segmentación, clustering aglomerativo, resegmentación y evaluación en las campañas NIST RT.
- Relevancia para DASR: Contexto histórico del pipeline en cascada que el PPT describe en la diapositiva 8.
- Etiquetas: [solapamiento: n/a] [nº hablantes: n/a] [tiempo real: n/a] [código: n/a] [relevancia: baja]
- PDF: descarga manual (de pago o sin copia libre localizada); ver [`pdfs/INDICE.md`](pdfs/INDICE.md)

### 5.2 Revisión de Park et al.
- Referencia: Tae Jin Park, Naoyuki Kanda, Dimitrios Dimitriadis, Kyu J. Han, Shinji Watanabe, Shrikanth Narayanan (2022). "A Review of Speaker Diarization: Recent Advances with Deep Learning". Computer Speech & Language 72. arXiv: https://arxiv.org/abs/2101.09624.
- Qué hace: Revisión de la diarización con deep learning: embeddings, clustering, EEND, integración con ASR, datasets y retos.
- Relevancia para DASR: Revisión de referencia previa a powerset, DiariZen y los Speech LLM.
- Etiquetas: [solapamiento: n/a] [nº hablantes: n/a] [tiempo real: n/a] [código: n/a] [relevancia: media]
- PDF: [5.02-Park-2022-A-Review-of-Speaker-Diarization-Recent-Advances-with.pdf](pdfs/1-diarizacion/5.02-Park-2022-A-Review-of-Speaker-Diarization-Recent-Advances-with.pdf)

### 5.3 Revisión experimental de Serafini et al.
- Referencia: Luca Serafini, Samuele Cornell, Giovanni Morrone, Enrico Zovato, Alessio Brutti, Stefano Squartini (2023). "An Experimental Review of Speaker Diarization methods with application to Two-Speaker Conversational Telephone Speech recordings". arXiv. arXiv: https://arxiv.org/abs/2305.18074.
- Qué hace: Compara ocho algoritmos (clustering, EEND y diarización guiada por separación) en cuatro corpus telefónicos de distintos idiomas, midiendo precisión y coste de inferencia. EEND-VC ofrece el mejor compromiso, pero los métodos EEND necesitan muchos datos anotados.
- Relevancia para DASR: Ejemplo de comparación justa entre familias de métodos, incluido el coste computacional; útil para diseñar la evaluación de DASR.
- Etiquetas: [solapamiento: n/a] [nº hablantes: n/a] [tiempo real: n/a] [código: n/a] [relevancia: media]
- PDF: [5.03-Serafini-2023-An-Experimental-Review-of-Speaker-Diarization-methods-with.pdf](pdfs/1-diarizacion/5.03-Serafini-2023-An-Experimental-Review-of-Speaker-Diarization-methods-with.pdf)

### 5.4 Benchmarking Diarization Models
- Referencia: Luca A. Lanzendörfer, Florian Grötschla, Cesare Blaser, Roger Wattenhofer (2025). "Benchmarking Diarization Models". arXiv. arXiv: https://arxiv.org/abs/2509.26177.
- Qué hace: Evalúa cinco sistemas sobre cuatro datasets (196.6 h en inglés, mandarín, alemán, japonés y español). PyannoteAI obtiene el mejor DER (11.2 %) y DiariZen es la mejor alternativa abierta (13.3 %). El error principal es el habla no detectada, seguido de la confusión de hablantes cuando hay muchos.
- Relevancia para DASR: La comparación independiente más reciente; incluye español y sitúa a pyannote frente a sus alternativas.
- Etiquetas: [solapamiento: n/a] [nº hablantes: n/a] [tiempo real: n/a] [código: n/a] [relevancia: alta]
- PDF: [5.04-Lanzendorfer-2025-Benchmarking-Diarization-Models.pdf](pdfs/1-diarizacion/5.04-Lanzendorfer-2025-Benchmarking-Diarization-Models.pdf)

### 5.5 On the Limitations of Speaker Diarization
- Referencia: Joana Amorim, João Pimentel, Frank Rudzicz (2026). "On the Limitations of Speaker Diarization". Expert Systems 43(3), e70221. DOI: https://doi.org/10.1111/exsy.70221.
- Qué hace: Evalúa cinco pipelines abiertos, de solo diarización y conjuntos con ASR, sobre datasets heterogéneos. Los dos tipos rinden parecido en escenarios no supervisados, dar el número de hablantes no mejora de forma consistente, y la poca duración del audio y la baja relación voz-ruido son lo que más perjudica. Recomienda mejores representaciones, reducir el ruido y ajustar hiperparámetros como la ventana y los umbrales de voz.
- Relevancia para DASR: Respalda la cascada de DASR y orienta el análisis de errores (por SNR y duración) y el ajuste de hiperparámetros.
- Etiquetas: [solapamiento: n/a] [nº hablantes: n/a] [tiempo real: n/a] [código: n/a] [acceso: abierto] [relevancia: alta]
- PDF: descarga manual (acceso abierto, pero Wiley bloquea las descargas automáticas: bajarlo desde el navegador); ver [`pdfs/INDICE.md`](pdfs/INDICE.md)

## Respuesta directa

Los mejores sistemas abiertos de diarización siguen el esquema local + global que reproduce DASR: una red segmenta ventanas cortas y detecta solapamiento, y un clustering da identidad global (pyannote 3.1 y community-1, DiariZen, EEND-VC, diart). La alternativa es la línea end-to-end (EEND, Sortformer, Nemotron 3), que resuelve la permutación dentro de la red y limita el número de hablantes. En la comparación independiente de 2025 el mejor sistema abierto es DiariZen (13.3 % de DER), que es precisamente pyannote con WavLM en la segmentación y VBx en el clustering. Por eso la base de DASR no está desfasada: las mejoras recientes cambian piezas del pipeline, no su arquitectura.

## Huecos detectados

1. No hay una comparación publicada entre AHC con enlace de centroides y VBx sobre los mismos embeddings locales de pyannote; el CHANGELOG de la 4.0 solo informa del cambio y la mejora global.
2. Las evaluaciones en español son escasas: el benchmark de 2025 incluye español, pero no separa los resultados de pyannote 3.1 en ese idioma.
3. Ningún trabajo documenta paso a paso, con tensores intermedios verificables, la reproducción de pyannote 3.1; el tutorial de DiariZen es lo más parecido.
4. La sensibilidad del sistema a la duración del audio y al ruido está identificada (Amorim et al. 2026), pero no medida para la configuración concreta de pyannote 3.1.
