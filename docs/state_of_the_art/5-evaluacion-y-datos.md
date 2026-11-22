# Frente 5: evaluación, métricas y datos

Revisión bibliográfica para DASR: con qué métricas y sobre qué datos se evalúa cada parte de DASR, incluido el español. Fecha de la búsqueda: 12 de noviembre de 2026. Autores, año, título y enlace de cada referencia se comprobaron contra arXiv o Crossref. Las venues que en arXiv solo constan como "enviado" o que no se pudieron confirmar van marcadas "(no verificado)". El resumen de cada trabajo se basa en su abstract, salvo cuando se indica otra cosa.

Leyenda de etiquetas: [tipo] métrica, dataset o reto, [idioma], [acceso] libre o con licencia, [relevancia] para DASR.

Los PDF de este banco están en [`pdfs/5-evaluacion-y-datos/`](pdfs/5-evaluacion-y-datos/); el [índice](pdfs/INDICE.md) lista los que hay que bajar a mano.

## 1. Métricas de diarización

### 1.1 pyannote.metrics
- Referencia: Hervé Bredin (2017). "pyannote.metrics: A Toolkit for Reproducible Evaluation, Diagnostic, and Error Analysis of Speaker Diarization Systems". Interspeech 2017. DOI: https://doi.org/10.21437/interspeech.2017-411.
- Qué hace: Toolkit para evaluar de forma reproducible la diarización y sus subtareas: DER con y sin collar, pureza, cobertura, JER y diagnóstico de errores.
- Relevancia para DASR: Herramienta de evaluación de la diarización de DASR; implementa el DER tal como lo reporta pyannote.
- Etiquetas: [tipo: métrica] [idioma: n/a] [acceso: libre; https://github.com/pyannote/pyannote-metrics] [relevancia: alta]
- PDF: [1.01-Bredin-2017-pyannote-metrics-A-Toolkit-for-Reproducible-Evaluation-Diagnostic.pdf](pdfs/5-evaluacion-y-datos/1.01-Bredin-2017-pyannote-metrics-A-Toolkit-for-Reproducible-Evaluation-Diagnostic.pdf)

### 1.2 DIHARD II y JER
- Referencia: Neville Ryant, Kenneth Church, Christopher Cieri, Alejandrina Cristia, Jun Du, Sriram Ganapathy, Mark Liberman (2019). "The Second DIHARD Diarization Challenge: Dataset, task, and baselines". Interspeech 2019. arXiv: https://arxiv.org/abs/1906.07839.
- Qué hace: Segundo reto DIHARD: datos de dominios difíciles, tareas y baselines; introduce el JER (Jaccard error rate) junto al DER. Su herramienta de evaluación es dscore.
- Relevancia para DASR: Fuente de la métrica JER que el PPT propone reportar junto al DER.
- Etiquetas: [tipo: métrica y reto] [idioma: inglés y otros] [acceso: datos con licencia LDC; dscore libre en https://github.com/nryant/dscore] [relevancia: media]
- PDF: [1.02-Ryant-2019-The-Second-DIHARD-Diarization-Challenge-Dataset-task-and.pdf](pdfs/5-evaluacion-y-datos/1.02-Ryant-2019-The-Second-DIHARD-Diarization-Challenge-Dataset-task-and.pdf)

## 2. Corpus de diarización

### 2.1 AMI
- Referencia: Jean Carletta, Simone Ashby, Sebastien Bourban, Mike Flynn, Mael Guillemot, Thomas Hain, Jaroslav Kadlec, Vasilis Karaiskos et al. (2006). "The AMI Meeting Corpus: A Pre-announcement". MLMI 2005, LNCS 3869. DOI: https://doi.org/10.1007/11677482_3.
- Qué hace: Corpus de 100 horas de reuniones en inglés con varios micrófonos, vídeo y anotaciones.
- Relevancia para DASR: Es el corpus que propone el plan del PPT para validar la reproducción (un subconjunto pequeño).
- Etiquetas: [tipo: dataset] [idioma: inglés] [acceso: libre; https://groups.inf.ed.ac.uk/ami/corpus/] [relevancia: alta]
- PDF: descarga manual (de pago o sin copia libre localizada); ver [`pdfs/INDICE.md`](pdfs/INDICE.md)

### 2.2 DIHARD III
- Referencia: Neville Ryant, Prachi Singh, Venkat Krishnamohan, Rajat Varma, Kenneth Church, Christopher Cieri, Jun Du, Sriram Ganapathy et al. (2020). "The Third DIHARD Diarization Challenge". arXiv. arXiv: https://arxiv.org/abs/2012.01477.
- Qué hace: Tercer reto DIHARD con once dominios difíciles (clínico, restaurante, YouTube, entre otros).
- Relevancia para DASR: Benchmark exigente en el que pyannote reporta su DER (tabla de la diapositiva 27 del PPT v2).
- Etiquetas: [tipo: dataset y reto] [idioma: inglés y otros] [acceso: licencia LDC] [relevancia: media]
- PDF: [2.02-Ryant-2020-The-Third-DIHARD-Diarization-Challenge.pdf](pdfs/5-evaluacion-y-datos/2.02-Ryant-2020-The-Third-DIHARD-Diarization-Challenge.pdf)

### 2.3 VoxConverse
- Referencia: Joon Son Chung, Jaesung Huh, Arsha Nagrani, Triantafyllos Afouras, Andrew Zisserman (2020). "Spot the conversation: speaker diarisation in the wild". Interspeech 2020. arXiv: https://arxiv.org/abs/2007.01216. DOI: https://doi.org/10.21437/Interspeech.2020-2337 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Corpus audiovisual de diarización con debates y noticias de YouTube, con muchos hablantes y solapamiento.
- Relevancia para DASR: Alternativa libre a DIHARD para evaluar DASR; DiariZen obtiene aquí su mejor resultado en el benchmark de 2025.
- Etiquetas: [tipo: dataset] [idioma: inglés principalmente] [acceso: libre; https://github.com/joonson/voxconverse] [relevancia: alta]
- PDF: [2.03-Chung-2020-Spot-the-conversation-speaker-diarisation-in-the-wild.pdf](pdfs/5-evaluacion-y-datos/2.03-Chung-2020-Spot-the-conversation-speaker-diarisation-in-the-wild.pdf)

### 2.4 AISHELL-4
- Referencia: Yihui Fu, Luyao Cheng, Shubo Lv, Yukai Jv, Yuxiang Kong, Zhuo Chen, Yanxin Hu, Lei Xie et al. (2021). "AISHELL-4: An Open Source Dataset for Speech Enhancement, Separation, Recognition and Speaker Diarization in Conference Scenario". Interspeech 2021. arXiv: https://arxiv.org/abs/2104.03603.
- Qué hace: Corpus de reuniones en mandarín con arreglo de 8 micrófonos para mejora, separación, reconocimiento y diarización.
- Relevancia para DASR: Benchmark en el que pyannote 3.1 reporta 12.2 % de DER.
- Etiquetas: [tipo: dataset] [idioma: mandarín] [acceso: libre (OpenSLR)] [relevancia: baja]
- PDF: [2.04-Fu-2021-AISHELL-4-An-Open-Source-Dataset-for-Speech.pdf](pdfs/5-evaluacion-y-datos/2.04-Fu-2021-AISHELL-4-An-Open-Source-Dataset-for-Speech.pdf)

### 2.5 AliMeeting (M2MeT)
- Referencia: Fan Yu, Shiliang Zhang, Yihui Fu, Lei Xie, Siqi Zheng, Zhihao Du, Weilong Huang, Pengcheng Guo et al. (2021). "M2MeT: The ICASSP 2022 Multi-Channel Multi-Party Meeting Transcription Challenge". ICASSP 2022. arXiv: https://arxiv.org/abs/2110.07393.
- Qué hace: Reto de transcripción de reuniones multicanal y multipartita con el corpus AliMeeting.
- Relevancia para DASR: Benchmark en el que la versión 4.0 de pyannote más mejora sobre la 3.1.
- Etiquetas: [tipo: dataset y reto] [idioma: mandarín] [acceso: libre (OpenSLR); https://github.com/yufan-aslp/AliMeeting] [relevancia: baja]
- PDF: [2.05-Yu-2021-M2MeT-The-ICASSP-2022-Multi-Channel-Multi-Party.pdf](pdfs/5-evaluacion-y-datos/2.05-Yu-2021-M2MeT-The-ICASSP-2022-Multi-Channel-Multi-Party.pdf)

### 2.6 MSDWild
- Referencia: Tao Liu, Shuai Fan, Xu Xiang, Hongbo Song, Shaoxiong Lin, Jiaqi Sun, Tianyuan Han, Siyuan Chen et al. (2022). "MSDWild: Multi-modal Speaker Diarization Dataset in the Wild". Interspeech 2022. DOI: https://doi.org/10.21437/interspeech.2022-10466.
- Qué hace: Dataset multimodal de diarización con vídeos "en la naturaleza", varios idiomas y conversaciones espontáneas.
- Relevancia para DASR: Opción de datos variados y libres para probar la generalización de DASR.
- Etiquetas: [tipo: dataset] [idioma: varios] [acceso: libre; https://github.com/X-LANCE/MSDWILD] [relevancia: baja]
- PDF: [2.06-Liu-2022-MSDWild-Multi-modal-Speaker-Diarization-Dataset-in-the.pdf](pdfs/5-evaluacion-y-datos/2.06-Liu-2022-MSDWild-Multi-modal-Speaker-Diarization-Dataset-in-the.pdf)

### 2.7 AVA-AVD
- Referencia: Eric Zhongcong Xu, Zeyang Song, Satoshi Tsutsui, Chao Feng, Mang Ye, Mike Zheng Shou (2021). "AVA-AVD: Audio-Visual Speaker Diarization in the Wild". ACM Multimedia 2022. arXiv: https://arxiv.org/abs/2111.14448. DOI: https://doi.org/10.1145/3503161.3548027 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Diarización audiovisual en películas, con muchos hablantes fuera de cámara.
- Relevancia para DASR: El benchmark más difícil de la tabla de pyannote (49.7 % de DER con 3.1).
- Etiquetas: [tipo: dataset] [idioma: varios] [acceso: libre; https://github.com/zcxu-eric/AVA-AVD] [relevancia: baja]
- PDF: [2.07-Xu-2021-AVA-AVD-Audio-Visual-Speaker-Diarization-in-the.pdf](pdfs/5-evaluacion-y-datos/2.07-Xu-2021-AVA-AVD-Audio-Visual-Speaker-Diarization-in-the.pdf)

### 2.8 CALLHOME
- Referencia: Linguistic Data Consortium (2001). "2000 NIST Speaker Recognition Evaluation (CALLHOME)". LDC2001S97. https://catalog.ldc.upenn.edu/LDC2001S97.
- Qué hace: Conversaciones telefónicas de 2 a 7 hablantes en varios idiomas, incluido el español; benchmark clásico de diarización.
- Relevancia para DASR: Benchmark de la tabla de pyannote (28.5 % de DER con 3.1) e incluye español, pero es de pago.
- Etiquetas: [tipo: dataset] [idioma: varios, incluido español] [acceso: licencia LDC] [relevancia: media]
- PDF: no aplica (página web, sin PDF)

## 3. Corpus de hablantes

### 3.1 VoxCeleb1
- Referencia: Arsha Nagrani, Joon Son Chung, Andrew Zisserman (2017). "VoxCeleb: a large-scale speaker identification dataset". Interspeech 2017. arXiv: https://arxiv.org/abs/1706.08612. DOI: https://doi.org/10.21437/Interspeech.2017-950 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Corpus de más de 100 000 frases de 1251 celebridades extraídas de YouTube para identificación y verificación de hablantes.
- Relevancia para DASR: Benchmark estándar de embeddings; base del protocolo de verificación (EER).
- Etiquetas: [tipo: dataset] [idioma: varios, mayoría inglés] [acceso: libre para investigación] [relevancia: media]
- PDF: [3.01-Nagrani-2017-VoxCeleb-a-large-scale-speaker-identification-dataset.pdf](pdfs/5-evaluacion-y-datos/3.01-Nagrani-2017-VoxCeleb-a-large-scale-speaker-identification-dataset.pdf)

### 3.2 VoxCeleb2
- Referencia: Joon Son Chung, Arsha Nagrani, Andrew Zisserman (2018). "VoxCeleb2: Deep Speaker Recognition". Interspeech 2018. arXiv: https://arxiv.org/abs/1806.05622. DOI: https://doi.org/10.21437/Interspeech.2018-1929 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Amplía VoxCeleb a más de un millón de frases de unas 6000 personas; es el conjunto con el que se entrenan la mayoría de los embeddings actuales.
- Relevancia para DASR: Datos de entrenamiento del ResNet34 de la Fase 2; explica el sesgo de dominio de los embeddings.
- Etiquetas: [tipo: dataset] [idioma: varios, mayoría inglés] [acceso: libre para investigación] [relevancia: media]
- PDF: [3.02-Chung-2018-VoxCeleb2-Deep-Speaker-Recognition.pdf](pdfs/5-evaluacion-y-datos/3.02-Chung-2018-VoxCeleb2-Deep-Speaker-Recognition.pdf)

## 4. Datos en español

### 4.1 Albayzin 2018 (IberSPEECH-RTVE)
- Referencia: Eduardo Lleida, Alfonso Ortega, Antonio Miguel, Virginia Bazán-Gil, Carmen Pérez, Manuel Gómez, Alberto de Prada (2019). "Albayzin 2018 Evaluation: The IberSpeech-RTVE Challenge on Speech Technologies for Spanish Broadcast Media". Applied Sciences 9(24). DOI: https://doi.org/10.3390/app9245412.
- Qué hace: Describe la evaluación Albayzin 2018 sobre la base de datos RTVE2018 de programas de televisión española, con retos de transcripción, diarización e identificación multimodal. RTVE sigue organizando retos de diarización (el último en IberSPEECH 2024).
- Relevancia para DASR: La fuente más relevante de datos anotados de diarización en español; requiere solicitar licencia.
- Etiquetas: [tipo: dataset y reto] [idioma: español] [acceso: licencia gratuita para investigación; https://catedrartve.unizar.es/rtvedatabase.html] [relevancia: alta]
- PDF: descarga manual (acceso abierto, pero MDPI bloquea las descargas automáticas: bajarlo desde el navegador); ver [`pdfs/INDICE.md`](pdfs/INDICE.md)

## Respuesta directa

Para la diarización, la métrica estándar es el DER calculado como lo hace pyannote (sin collar y contando solapamiento), con `pyannote.metrics`, más el JER. Para el ASR, WER con `jiwer` y, para la salida conjunta, cpWER o tcpWER con MeetEval. Los datos libres más útiles son AMI y VoxConverse; los benchmarks de pago (DIHARD III, CALLHOME) solo sirven para comparar con cifras publicadas. En español, la opción seria es RTVE (con licencia) más grabaciones propias anotadas.

## Huecos detectados

1. No hay un benchmark libre de diarización en español comparable a VoxConverse; RTVE requiere licencia.
2. Los benchmarks de diarización no reportan resultados por condición (SNR, duración, número de hablantes) de forma estandarizada.
3. No existe un protocolo común para evaluar la cadena completa diarización + ASR + atributos.
