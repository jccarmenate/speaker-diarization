# Frente 2: representaciones de hablante y clustering

Revisión bibliográfica para DASR: qué embeddings y qué algoritmos de clustering se usan para dar identidad global a los hablantes. Fecha de la búsqueda: 12 de noviembre de 2026. Autores, año, título y enlace de cada referencia se comprobaron contra arXiv o Crossref. Las venues que en arXiv solo constan como "enviado" o que no se pudieron confirmar van marcadas "(no verificado)". El resumen de cada trabajo se basa en su abstract, salvo cuando se indica otra cosa.

Leyenda de etiquetas: [tipo] embedding, autosupervisado o clustering, [usado en] dónde aparece dentro de DASR, [código], [relevancia] para DASR.

Los PDF de este banco están en [`pdfs/2-representaciones-y-clustering/`](pdfs/2-representaciones-y-clustering/); el [índice](pdfs/INDICE.md) lista los que hay que bajar a mano.

## 1. Embeddings de hablante

### 1.1 i-vectors
- Referencia: Najim Dehak, Patrick J Kenny, Réda Dehak, Pierre Dumouchel, Pierre Ouellet (2011). "Front-End Factor Analysis for Speaker Verification". IEEE TASLP 19(4). DOI: https://doi.org/10.1109/tasl.2010.2064307.
- Qué hace: Representa cada grabación con un vector de baja dimensión obtenido por análisis factorial sobre un modelo GMM-UBM, comparado con coseno o PLDA.
- Relevancia para DASR: Base histórica de la diarización por clustering de embeddings.
- Etiquetas: [tipo: embedding estadístico] [usado en: contexto] [código: n/a] [relevancia: baja]
- PDF: descarga manual (de pago o sin copia libre localizada); ver [`pdfs/INDICE.md`](pdfs/INDICE.md)

### 1.2 x-vectors
- Referencia: David Snyder, Daniel Garcia-Romero, Gregory Sell, Daniel Povey, Sanjeev Khudanpur (2018). "X-Vectors: Robust DNN Embeddings for Speaker Recognition". ICASSP 2018. DOI: https://doi.org/10.1109/icassp.2018.8461375.
- Qué hace: Red TDNN con statistics pooling entrenada para clasificar hablantes, con aumento de datos (ruido y reverberación). El vector de una capa intermedia es el embedding y supera a los i-vectors.
- Relevancia para DASR: Los embeddings sobre los que se definieron VBx y el clustering clásico de diarización.
- Etiquetas: [tipo: embedding neuronal] [usado en: contexto de la Fase 2] [código: Kaldi (no verificado)] [relevancia: media]
- PDF: [1.02-Snyder-2018-X-Vectors-Robust-DNN-Embeddings-for-Speaker-Recognition.pdf](pdfs/2-representaciones-y-clustering/1.02-Snyder-2018-X-Vectors-Robust-DNN-Embeddings-for-Speaker-Recognition.pdf)

### 1.3 SincNet
- Referencia: Mirco Ravanelli, Yoshua Bengio (2018). "Speaker Recognition from Raw Waveform with SincNet". SLT 2018. arXiv: https://arxiv.org/abs/1808.00158.
- Qué hace: Primera capa convolucional con filtros paso banda paramétricos (funciones sinc) aprendidos directamente sobre la forma de onda: solo aprende las frecuencias de corte de cada filtro.
- Relevancia para DASR: Es la primera capa de PyanNet, la red de segmentación de la Fase 1 de DASR.
- Etiquetas: [tipo: capa de entrada] [usado en: Fase 1] [código: https://github.com/mravanelli/SincNet] [relevancia: alta]
- PDF: [1.03-Ravanelli-2018-Speaker-Recognition-from-Raw-Waveform-with-SincNet.pdf](pdfs/2-representaciones-y-clustering/1.03-Ravanelli-2018-Speaker-Recognition-from-Raw-Waveform-with-SincNet.pdf)

### 1.4 Margin Matters
- Referencia: Xu Xiang, Shuai Wang, Houjun Huang, Yanmin Qian, Kai Yu (2019). "Margin Matters: Towards More Discriminative Deep Neural Network Embeddings for Speaker Recognition". arXiv. arXiv: https://arxiv.org/abs/1906.07317.
- Qué hace: Estudia pérdidas de margen (AM-softmax, AAM-softmax) para entrenar embeddings de hablante más discriminativos.
- Relevancia para DASR: Explica la familia de pérdidas con la que se entrenan los embeddings modernos que usa la Fase 2.
- Etiquetas: [tipo: pérdida de entrenamiento] [usado en: Fase 2] [código: n/a] [relevancia: baja]
- PDF: [1.04-Xiang-2019-Margin-Matters-Towards-More-Discriminative-Deep-Neural-Network.pdf](pdfs/2-representaciones-y-clustering/1.04-Xiang-2019-Margin-Matters-Towards-More-Discriminative-Deep-Neural-Network.pdf)

### 1.5 In defence of metric learning
- Referencia: Joon Son Chung, Jaesung Huh, Seongkyu Mun, Minjae Lee, Hee Soo Heo, Soyeon Choe, Chiheon Ham, Sunghwan Jung et al. (2020). "In defence of metric learning for speaker recognition". Interspeech 2020. arXiv: https://arxiv.org/abs/2003.11982. DOI: https://doi.org/10.21437/Interspeech.2020-1064 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Compara pérdidas de clasificación y de metric learning para reconocimiento de hablante en VoxCeleb con un protocolo común y publica el entrenador voxceleb_trainer.
- Relevancia para DASR: Referencia práctica de cómo se entrenan y evalúan embeddings con VoxCeleb.
- Etiquetas: [tipo: embedding neuronal] [usado en: contexto] [código: https://github.com/clovaai/voxceleb_trainer] [relevancia: baja]
- PDF: [1.05-Chung-2020-In-defence-of-metric-learning-for-speaker-recognition.pdf](pdfs/2-representaciones-y-clustering/1.05-Chung-2020-In-defence-of-metric-learning-for-speaker-recognition.pdf)

### 1.6 ECAPA-TDNN
- Referencia: Brecht Desplanques, Jenthe Thienpondt, Kris Demuynck (2020). "ECAPA-TDNN: Emphasized Channel Attention, Propagation and Aggregation in TDNN Based Speaker Verification". Interspeech 2020. arXiv: https://arxiv.org/abs/2005.07143. DOI: https://doi.org/10.21437/Interspeech.2020-2650 Focus to learn more DOI(s) linking to related resources.
- Qué hace: TDNN con atención por canal (squeeze-excitation), agregación multicapa y attentive statistics pooling. Fue el estado del arte en VoxCeleb y es el embedding por defecto de SpeechBrain.
- Relevancia para DASR: Alternativa directa a ResNet34 en la Fase 2 y candidata para las sondas de AVA.
- Etiquetas: [tipo: embedding neuronal] [usado en: Fase 2 (alternativa), AVA] [código: https://github.com/speechbrain/speechbrain] [relevancia: media]
- PDF: [1.06-Desplanques-2020-ECAPA-TDNN-Emphasized-Channel-Attention-Propagation-and-Aggregation.pdf](pdfs/2-representaciones-y-clustering/1.06-Desplanques-2020-ECAPA-TDNN-Emphasized-Channel-Attention-Propagation-and-Aggregation.pdf)

### 1.7 TitaNet
- Referencia: Nithin Rao Koluguri, Taejin Park, Boris Ginsburg (2021). "TitaNet: Neural Model for speaker representation with 1D Depth-wise separable convolutions and global context". ICASSP 2022 (no verificado). arXiv: https://arxiv.org/abs/2110.04410.
- Qué hace: Modelo de embeddings de NVIDIA con convoluciones separables 1D y contexto global; es el embedding de los pipelines de diarización de NeMo.
- Relevancia para DASR: Equivalente en NeMo del embedding de la Fase 2.
- Etiquetas: [tipo: embedding neuronal] [usado en: contexto] [código: https://github.com/NVIDIA-NeMo/Speech] [relevancia: baja]
- PDF: [1.07-Koluguri-2021-TitaNet-Neural-Model-for-speaker-representation-with-1D.pdf](pdfs/2-representaciones-y-clustering/1.07-Koluguri-2021-TitaNet-Neural-Model-for-speaker-representation-with-1D.pdf)

### 1.8 WeSpeaker
- Referencia: Hongji Wang, Chengdong Liang, Shuai Wang, Zhengyang Chen, Binbin Zhang, Xu Xiang, Yanlei Deng, Yanmin Qian (2023). "Wespeaker: A Research and Production oriented Speaker Embedding Learning Toolkit". ICASSP 2023. arXiv: https://arxiv.org/abs/2210.17016.
- Qué hace: Toolkit de embeddings de hablante para investigación y producción, con recetas de ResNet, ECAPA y otras arquitecturas y modelos preentrenados en VoxCeleb.
- Relevancia para DASR: De aquí sale el ResNet34 de 256 dimensiones que usa pyannote 3.1 (`pyannote/wespeaker-voxceleb-resnet34-LM`), el modelo de embedding de la Fase 2.
- Etiquetas: [tipo: embedding neuronal] [usado en: Fase 2] [código: https://github.com/wenet-e2e/wespeaker] [relevancia: alta]
- PDF: [1.08-Wang-2023-Wespeaker-A-Research-and-Production-oriented-Speaker-Embedding.pdf](pdfs/2-representaciones-y-clustering/1.08-Wang-2023-Wespeaker-A-Research-and-Production-oriented-Speaker-Embedding.pdf)

### 1.9 CAM++
- Referencia: Hui Wang, Siqi Zheng, Yafeng Chen, Luyao Cheng, Qian Chen (2023). "CAM++: A Fast and Efficient Network for Speaker Verification Using Context-Aware Masking". Interspeech 2023 (no verificado). arXiv: https://arxiv.org/abs/2303.00332.
- Qué hace: D-TDNN con context-aware masking; reporta menos parámetros y menor EER que ECAPA-TDNN en VoxCeleb1-O.
- Relevancia para DASR: Embedding eficiente alternativo para la Fase 2 o para AVA.
- Etiquetas: [tipo: embedding neuronal] [usado en: alternativa] [código: https://github.com/modelscope/3D-Speaker] [relevancia: baja]
- PDF: [1.09-Wang-2023-CAM-A-Fast-and-Efficient-Network-for-Speaker.pdf](pdfs/2-representaciones-y-clustering/1.09-Wang-2023-CAM-A-Fast-and-Efficient-Network-for-Speaker.pdf)

### 1.10 ReDimNet
- Referencia: Ivan Yakovlev, Rostislav Makarov, Andrei Balykin, Pavel Malov, Anton Okhotnikov, Nikita Torgashov (2024). "Reshape Dimensions Network for Speaker Recognition". Interspeech 2024. arXiv: https://arxiv.org/abs/2407.18223. DOI: https://doi.org/10.21437/Interspeech.2024-2116 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Arquitectura que alterna bloques 1D y 2D remodelando las dimensiones; resultados de primera línea en VoxCeleb con pocos parámetros.
- Relevancia para DASR: Candidato reciente si se quisiera mejorar el embedding de la Fase 2.
- Etiquetas: [tipo: embedding neuronal] [usado en: alternativa] [código: https://github.com/IDRnD/redimnet] [relevancia: baja]
- PDF: [1.10-Yakovlev-2024-Reshape-Dimensions-Network-for-Speaker-Recognition.pdf](pdfs/2-representaciones-y-clustering/1.10-Yakovlev-2024-Reshape-Dimensions-Network-for-Speaker-Recognition.pdf)

## 2. Representaciones autosupervisadas

### 2.1 wav2vec 2.0
- Referencia: Alexei Baevski, Henry Zhou, Abdelrahman Mohamed, Michael Auli (2020). "wav2vec 2.0: A Framework for Self-Supervised Learning of Speech Representations". NeurIPS 2020. arXiv: https://arxiv.org/abs/2006.11477.
- Qué hace: Preentrenamiento autosupervisado: enmascara representaciones latentes del audio y resuelve una tarea contrastiva sobre unidades cuantizadas. Afinado con CTC logra ASR con muy pocos datos etiquetados.
- Relevancia para DASR: Base de la alineación forzada de la rama ASR (WhisperX usa modelos wav2vec 2.0 con CTC) y de varios modelos de atributos de AVA.
- Etiquetas: [tipo: autosupervisado] [usado en: rama ASR (FA), AVA] [código: https://github.com/facebookresearch/fairseq (archivado)] [relevancia: alta]
- PDF: [2.01-Baevski-2020-wav2vec-2-0-A-Framework-for-Self-Supervised.pdf](pdfs/2-representaciones-y-clustering/2.01-Baevski-2020-wav2vec-2-0-A-Framework-for-Self-Supervised.pdf)

### 2.2 HuBERT
- Referencia: Wei-Ning Hsu, Benjamin Bolte, Yao-Hung Hubert Tsai, Kushal Lakhotia, Ruslan Salakhutdinov, Abdelrahman Mohamed (2021). "HuBERT: Self-Supervised Speech Representation Learning by Masked Prediction of Hidden Units". IEEE/ACM TASLP 2021 (no verificado). arXiv: https://arxiv.org/abs/2106.07447.
- Qué hace: Predicción enmascarada de unidades ocultas obtenidas por clustering iterativo de rasgos acústicos.
- Relevancia para DASR: Antecesor directo de WavLM.
- Etiquetas: [tipo: autosupervisado] [usado en: contexto] [código: https://github.com/facebookresearch/fairseq (archivado)] [relevancia: baja]
- PDF: [2.02-Hsu-2021-HuBERT-Self-Supervised-Speech-Representation-Learning-by-Masked.pdf](pdfs/2-representaciones-y-clustering/2.02-Hsu-2021-HuBERT-Self-Supervised-Speech-Representation-Learning-by-Masked.pdf)

### 2.3 WavLM
- Referencia: Sanyuan Chen, Chengyi Wang, Zhengyang Chen, Yu Wu, Shujie Liu, Zhuo Chen, Jinyu Li, Naoyuki Kanda et al. (2022). "WavLM: Large-Scale Self-Supervised Pre-Training for Full Stack Speech Processing". IEEE JSTSP 16(6). arXiv: https://arxiv.org/abs/2110.13900. DOI: https://doi.org/10.1109/JSTSP.2022.3188113 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Extiende HuBERT con denoising (mezcla voces y ruido durante el preentrenamiento) para cubrir todo el espectro de tareas del habla, incluidas la diarización y la verificación de hablantes.
- Relevancia para DASR: Es el codificador de DiariZen [B1 §4.1] y la representación que la literatura de atributos señala como más informativa [B4 §1.2, §2.5].
- Etiquetas: [tipo: autosupervisado] [usado en: extensión de la Fase 1, AVA] [código: https://github.com/microsoft/unilm] [relevancia: alta]
- PDF: [2.03-Chen-2022-WavLM-Large-Scale-Self-Supervised-Pre-Training-for.pdf](pdfs/2-representaciones-y-clustering/2.03-Chen-2022-WavLM-Large-Scale-Self-Supervised-Pre-Training-for.pdf)

### 2.4 SUPERB
- Referencia: Shu-wen Yang, Po-Han Chi, Yung-Sung Chuang, Cheng-I Jeff Lai, Kushal Lakhotia, Yist Y. Lin, Andy T. Liu, Jiatong Shi et al. (2021). "SUPERB: Speech processing Universal PERformance Benchmark". Interspeech 2021. arXiv: https://arxiv.org/abs/2105.01051.
- Qué hace: Benchmark que evalúa representaciones autosupervisadas congeladas en muchas tareas del habla (hablante, diarización, ASR, emoción) con cabezas ligeras.
- Relevancia para DASR: Es el protocolo de "sondeo" de representaciones que AVA puede imitar.
- Etiquetas: [tipo: benchmark] [usado en: AVA] [código: https://github.com/s3prl/s3prl] [relevancia: media]
- PDF: [2.04-Yang-2021-SUPERB-Speech-processing-Universal-PERformance-Benchmark.pdf](pdfs/2-representaciones-y-clustering/2.04-Yang-2021-SUPERB-Speech-processing-Universal-PERformance-Benchmark.pdf)

## 3. Clustering para diarización

### 3.1 PLDA y AHC con calibración
- Referencia: Gregory Sell, Daniel Garcia-Romero (2014). "Speaker diarization with plda i-vector scoring and unsupervised calibration". SLT 2014. DOI: https://doi.org/10.1109/slt.2014.7078610.
- Qué hace: Diarización con i-vectors puntuados con PLDA y clustering aglomerativo, con calibración no supervisada del umbral de parada.
- Relevancia para DASR: Antecedente del AHC con umbral de la Fase 3.
- Etiquetas: [tipo: clustering] [usado en: contexto de la Fase 3] [código: n/a] [relevancia: baja]
- PDF: descarga manual (de pago o sin copia libre localizada); ver [`pdfs/INDICE.md`](pdfs/INDICE.md)

### 3.2 Clustering espectral con d-vectors
- Referencia: Quan Wang, Carlton Downey, Li Wan, Philip Andrew Mansfield, Ignacio Lopez Moreno (2017). "Speaker Diarization with LSTM". ICASSP 2018. arXiv: https://arxiv.org/abs/1710.10468.
- Qué hace: Usa embeddings de una LSTM (d-vectors) y clustering espectral con varios refinamientos de la matriz de afinidad.
- Relevancia para DASR: Origen del clustering espectral moderno en diarización, una de las alternativas que el PPT descarta (diapositivas 16 y 17).
- Etiquetas: [tipo: clustering] [usado en: alternativa de la Fase 3] [código: https://github.com/wq2012/SpectralCluster] [relevancia: media]
- PDF: [3.02-Wang-2017-Speaker-Diarization-with-LSTM.pdf](pdfs/2-representaciones-y-clustering/3.02-Wang-2017-Speaker-Diarization-with-LSTM.pdf)

### 3.3 UIS-RNN
- Referencia: Aonan Zhang, Quan Wang, Zhenyao Zhu, John Paisley, Chong Wang (2018). "Fully Supervised Speaker Diarization". ICASSP 2019. arXiv: https://arxiv.org/abs/1810.04719.
- Qué hace: Clustering supervisado: una RNN genera la secuencia de hablantes y un proceso de Chinese restaurant admite hablantes nuevos.
- Relevancia para DASR: Alternativa supervisada; muestra el coste de necesitar datos etiquetados para agrupar.
- Etiquetas: [tipo: clustering supervisado] [usado en: contexto] [código: https://github.com/google/uis-rnn (archivado)] [relevancia: baja]
- PDF: [3.03-Zhang-2018-Fully-Supervised-Speaker-Diarization.pdf](pdfs/2-representaciones-y-clustering/3.03-Zhang-2018-Fully-Supervised-Speaker-Diarization.pdf)

### 3.4 NME-SC
- Referencia: Tae Jin Park, Kyu J. Han, Manoj Kumar, Shrikanth Narayanan (2020). "Auto-Tuning Spectral Clustering for Speaker Diarization Using Normalized Maximum Eigengap". IEEE Signal Processing Letters. arXiv: https://arxiv.org/abs/2003.02405. DOI: https://doi.org/10.1109/LSP.2019.2961071 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Clustering espectral que ajusta automáticamente la poda de la matriz de afinidad y estima el número de hablantes con la brecha de autovalores normalizada.
- Relevancia para DASR: Es el clustering espectral que usa NeMo y la variante que el PPT compara con AHC.
- Etiquetas: [tipo: clustering] [usado en: alternativa de la Fase 3] [código: https://github.com/tango4j/Auto-Tuning-Spectral-Clustering] [relevancia: media]
- PDF: [3.04-Park-2020-Auto-Tuning-Spectral-Clustering-for-Speaker-Diarization-Using.pdf](pdfs/2-representaciones-y-clustering/3.04-Park-2020-Auto-Tuning-Spectral-Clustering-for-Speaker-Diarization-Using.pdf)

### 3.5 VBx
- Referencia: Federico Landini, Ján Profant, Mireia Diez, Lukáš Burget (2022). "Bayesian HMM clustering of x-vector sequences (VBx) in speaker diarization: theory, implementation and analysis on standard tasks". Computer Speech & Language. arXiv: https://arxiv.org/abs/2012.14952.
- Qué hace: Modelo bayesiano: un HMM sobre la secuencia de x-vectors en el que cada estado es un hablante, con inferencia variacional y PLDA; normalmente se inicializa con AHC.
- Relevancia para DASR: La alternativa de clustering que discute el PPT; pyannote 4.0 la adoptó adaptada a los embeddings locales.
- Etiquetas: [tipo: clustering] [usado en: extensión de la Fase 3] [código: https://github.com/BUTSpeechFIT/VBx] [relevancia: alta]
- PDF: [3.05-Landini-2022-Bayesian-HMM-clustering-of-x-vector-sequences-VBx.pdf](pdfs/2-representaciones-y-clustering/3.05-Landini-2022-Bayesian-HMM-clustering-of-x-vector-sequences-VBx.pdf)

### 3.6 Algoritmos de clustering jerárquico
- Referencia: Daniel Müllner (2011). "Modern hierarchical, agglomerative clustering algorithms". arXiv. arXiv: https://arxiv.org/abs/1109.2378.
- Qué hace: Revisa los algoritmos de clustering jerárquico aglomerativo (single, complete, average, weighted, Ward, centroid, median), su complejidad y sus implementaciones eficientes. Documenta que los enlaces centroid y median pueden producir inversiones en el dendrograma.
- Relevancia para DASR: Es la base de `scipy.cluster.hierarchy.linkage`, que usa pyannote 3.1 en la Fase 3, y justifica la precaución sobre inversiones de la diapositiva 20.
- Etiquetas: [tipo: clustering] [usado en: Fase 3] [código: fastcluster; SciPy] [relevancia: alta]
- PDF: [3.06-Mullner-2011-Modern-hierarchical-agglomerative-clustering-algorithms.pdf](pdfs/2-representaciones-y-clustering/3.06-Mullner-2011-Modern-hierarchical-agglomerative-clustering-algorithms.pdf)

### 3.7 Algoritmo húngaro
- Referencia: H. W. Kuhn (1955). "The Hungarian method for the assignment problem". Naval Research Logistics Quarterly 2(1–2). DOI: https://doi.org/10.1002/nav.3800020109.
- Qué hace: Resuelve el problema de asignación de coste mínimo entre dos conjuntos del mismo tamaño.
- Relevancia para DASR: Es la asignación restringida por ventana de la Fase 3 (`scipy.optimize.linear_sum_assignment` dentro de `assign_embeddings`).
- Etiquetas: [tipo: asignación] [usado en: Fase 3] [código: SciPy] [relevancia: media]
- PDF: descarga manual (de pago o sin copia libre localizada); ver [`pdfs/INDICE.md`](pdfs/INDICE.md)

### 3.8 Clustering en verificación autosupervisada
- Referencia: Abderrahim Fathan, Jahangir Alam (2024). "An analytic study on clustering driven self-supervised speaker verification". Pattern Recognition Letters 179, 80–86. DOI: https://doi.org/10.1016/j.patrec.2024.01.024.
- Qué hace: En verificación de hablantes autosupervisada, compara pseudo-etiquetas generadas con distintos algoritmos de clustering y encuentra alta correlación entre las métricas de calidad del clustering y el rendimiento final (resumen tomado de una reseña; texto completo no consultado).
- Relevancia para DASR: Argumento para medir la calidad del clustering de la Fase 3 por sí misma (pureza, cobertura), además del DER.
- Etiquetas: [tipo: clustering] [usado en: evaluación de la Fase 3] [código: n/a] [relevancia: media]
- PDF: descarga manual (de pago o sin copia libre localizada); ver [`pdfs/INDICE.md`](pdfs/INDICE.md)

## Respuesta directa

La Fase 2 de DASR usa el estándar actual de pyannote: un ResNet34 de WeSpeaker de 256 dimensiones entrenado con VoxCeleb. Las alternativas maduras son ECAPA-TDNN, CAM++ y ReDimNet, y la tendencia es usar modelos autosupervisados (WavLM) como codificador. Para la Fase 3 conviven tres familias: AHC (pyannote 3.1), clustering espectral (NeMo) y VBx (pyannote 4.0, DiariZen). La elección de AHC con centroides está bien sustentada por su encaje con embeddings sin orden, pero la literatura reciente se ha movido a VBx.

## Huecos detectados

1. No hay estudios que comparen embeddings (ResNet34, ECAPA, WavLM) dentro de la misma configuración de pyannote 3.1 con su clustering de centroides.
2. No se encontró un análisis publicado de cuántas veces aparecen inversiones del dendrograma de centroides en diarización real, ni de su efecto en el DER.
3. El efecto del umbral δ y de min_cluster_size en audios en español no está documentado.
