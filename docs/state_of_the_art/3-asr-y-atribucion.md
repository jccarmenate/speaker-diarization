# Frente 3: ASR y atribución de palabras a hablantes

Revisión bibliográfica para DASR: cómo se obtiene el texto con tiempos por palabra y cómo se une con la diarización para saber quién dijo qué. Fecha de la búsqueda: 12 de noviembre de 2026. Autores, año, título y enlace de cada referencia se comprobaron contra arXiv o Crossref. Las venues que en arXiv solo constan como "enviado" o que no se pudieron confirmar van marcadas "(no verificado)". El resumen de cada trabajo se basa en su abstract, salvo cuando se indica otra cosa.

Leyenda de etiquetas: [tiempos por palabra] si da tiempos fiables por palabra, [integración] cascada o modelo conjunto, [código], [relevancia] para DASR.

Los PDF de este banco están en [`pdfs/3-asr-y-atribucion/`](pdfs/3-asr-y-atribucion/); el [índice](pdfs/INDICE.md) lista los que hay que bajar a mano.

## 1. ASR y alineación forzada

### 1.1 CTC
- Referencia: Alex Graves, Santiago Fernández, Faustino Gomez, Jürgen Schmidhuber (2006). "Connectionist temporal classification". ICML 2006. DOI: https://doi.org/10.1145/1143844.1143891.
- Qué hace: Define la pérdida CTC, que entrena redes recurrentes para etiquetar secuencias sin alineación previa sumando sobre todas las alineaciones posibles, con un símbolo "blanco".
- Relevancia para DASR: Fundamento de la alineación forzada de la rama ASR: con el texto conocido, el camino CTC más probable da los tiempos de cada palabra.
- Etiquetas: [tiempos por palabra: base del método] [integración: n/a] [código: n/a] [relevancia: alta]
- PDF: [1.01-Graves-2006-Connectionist-temporal-classification.pdf](pdfs/3-asr-y-atribucion/1.01-Graves-2006-Connectionist-temporal-classification.pdf)

### 1.2 Whisper
- Referencia: Alec Radford, Jong Wook Kim, Tao Xu, Greg Brockman, Christine McLeavey, Ilya Sutskever (2023). "Robust Speech Recognition via Large-Scale Weak Supervision". ICML 2023. arXiv: https://arxiv.org/abs/2212.04356.
- Qué hace: Encoder-decoder Transformer entrenado con 680 000 h de audio débilmente etiquetado, multilingüe y multitarea, con entrada log-Mel. Es robusto sin afinar, pero sus marcas de tiempo son por segmento.
- Relevancia para DASR: Es el ASR de la rama de DASR.
- Etiquetas: [tiempos por palabra: no fiables] [integración: n/a] [código: https://github.com/openai/whisper] [relevancia: alta]
- PDF: [1.02-Radford-2023-Robust-Speech-Recognition-via-Large-Scale-Weak-Supervision.pdf](pdfs/3-asr-y-atribucion/1.02-Radford-2023-Robust-Speech-Recognition-via-Large-Scale-Weak-Supervision.pdf)

### 1.3 WhisperX
- Referencia: Max Bain, Jaesung Huh, Tengda Han, Andrew Zisserman (2023). "WhisperX: Time-Accurate Speech Transcription of Long-Form Audio". Interspeech 2023. arXiv: https://arxiv.org/abs/2303.00747.
- Qué hace: Combina detección de voz, transcripción por lotes con Whisper y alineación forzada por fonemas con wav2vec 2.0 para obtener tiempos por palabra; asigna hablantes con la diarización de pyannote. Reduce las alucinaciones y acelera la transcripción de audio largo.
- Relevancia para DASR: Es la referencia directa de la rama ASR de DASR y de `Pipeline.assign_speakers`.
- Etiquetas: [tiempos por palabra: sí] [integración: cascada] [código: https://github.com/m-bain/whisperX] [relevancia: alta]
- PDF: [1.03-Bain-2023-WhisperX-Time-Accurate-Speech-Transcription-of-Long-Form.pdf](pdfs/3-asr-y-atribucion/1.03-Bain-2023-WhisperX-Time-Accurate-Speech-Transcription-of-Long-Form.pdf)

### 1.4 CTC-Segmentation
- Referencia: Ludwig Kürzinger, Dominik Winkelbauer, Lujun Li, Tobias Watzel, Gerhard Rigoll (2020). "CTC-Segmentation of Large Corpora for German End-to-end Speech Recognition". SPECOM 2020. arXiv: https://arxiv.org/abs/2007.09127. DOI: https://doi.org/10.1007/978-3-030-60276-5_27 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Alinea textos largos con su audio usando las probabilidades de un modelo CTC, para segmentar corpus grandes.
- Relevancia para DASR: Alternativa documentada de alineación para la rama ASR.
- Etiquetas: [tiempos por palabra: sí] [integración: n/a] [código: https://github.com/lumaku/ctc-segmentation] [relevancia: media]
- PDF: [1.04-Kurzinger-2020-CTC-Segmentation-of-Large-Corpora-for-German-End.pdf](pdfs/3-asr-y-atribucion/1.04-Kurzinger-2020-CTC-Segmentation-of-Large-Corpora-for-German-End.pdf)

### 1.5 MMS
- Referencia: Vineel Pratap, Andros Tjandra, Bowen Shi, Paden Tomasello, Arun Babu, Sayani Kundu, Ali Elkahky, Zhaoheng Ni et al. (2023). "Scaling Speech Technology to 1,000+ Languages". arXiv. arXiv: https://arxiv.org/abs/2305.13516.
- Qué hace: Modelos wav2vec 2.0 preentrenados para 1406 idiomas, ASR para 1107 idiomas e identificación de idioma para 4017.
- Relevancia para DASR: Fuente de modelos multilingües de alineación si los de WhisperX no bastan para español u otros idiomas.
- Etiquetas: [tiempos por palabra: con alineador CTC] [integración: n/a] [código: https://github.com/facebookresearch/fairseq (archivado)] [relevancia: baja]
- PDF: [1.05-Pratap-2023-Scaling-Speech-Technology-to-1-000-Languages.pdf](pdfs/3-asr-y-atribucion/1.05-Pratap-2023-Scaling-Speech-Technology-to-1-000-Languages.pdf)

## 2. ASR atribuido y modelos conjuntos

### 2.1 ASR y diarización con un transductor
- Referencia: Laurent El Shafey, Hagen Soltau, Izhak Shafran (2019). "Joint Speech Recognition and Speaker Diarization via Sequence Transduction". Interspeech 2019. arXiv: https://arxiv.org/abs/1907.05337.
- Qué hace: Un transductor RNN-T emite palabras y roles de hablante a la vez, usando pistas lingüísticas y acústicas, frente a la fusión ad hoc de dos sistemas independientes.
- Relevancia para DASR: Primer ejemplo influyente de modelo conjunto; señala el problema de las fronteras que DASR resuelve con la diarización exclusiva.
- Etiquetas: [tiempos por palabra: n/a] [integración: conjunto] [código: no localizado] [relevancia: baja]
- PDF: [2.01-Shafey-2019-Joint-Speech-Recognition-and-Speaker-Diarization-via-Sequence.pdf](pdfs/3-asr-y-atribucion/2.01-Shafey-2019-Joint-Speech-Recognition-and-Speaker-Diarization-via-Sequence.pdf)

### 2.2 ASR atribuido con SOT
- Referencia: Naoyuki Kanda, Yashesh Gaur, Xiaofei Wang, Zhong Meng, Zhuo Chen, Tianyan Zhou, Takuya Yoshioka (2020). "Joint Speaker Counting, Speech Recognition, and Speaker Identification for Overlapped Speech of Any Number of Speakers". Interspeech 2020. arXiv: https://arxiv.org/abs/2006.10930.
- Qué hace: Modelo de ASR atribuido a hablante que cuenta hablantes, transcribe e identifica hablantes en habla solapada, con un inventario de hablantes como entrada auxiliar.
- Relevancia para DASR: Referencia de ASR atribuido de punta a punta, en contraste con la cascada de DASR.
- Etiquetas: [tiempos por palabra: n/a] [integración: conjunto] [código: no localizado] [relevancia: baja]
- PDF: [2.02-Kanda-2020-Joint-Speaker-Counting-Speech-Recognition-and-Speaker-Identification.pdf](pdfs/3-asr-y-atribucion/2.02-Kanda-2020-Joint-Speaker-Counting-Speech-Recognition-and-Speaker-Identification.pdf)

### 2.3 DiarizationLM
- Referencia: Quan Wang, Yiling Huang, Guanlong Zhao, Evan Clark, Wei Xia, Hank Liao (2024). "DiarizationLM: Speaker Diarization Post-Processing with Large Language Models". Interspeech 2024, pp. 3754–3758. arXiv: https://arxiv.org/abs/2401.03506. DOI: https://doi.org/10.21437/Interspeech.2024-209 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Serializa la salida de ASR y diarización como texto y usa un LLM afinado para corregir las etiquetas de hablante; reduce la tasa de error de diarización por palabra sin reentrenar los sistemas base.
- Relevancia para DASR: Extensión aplicable sobre la salida de DASR sin tocar el pipeline.
- Etiquetas: [tiempos por palabra: usa los del ASR] [integración: post-procesado] [código: https://github.com/google/speaker-id (no verificado que incluya DiarizationLM)] [relevancia: media]
- PDF: [2.03-Wang-2024-DiarizationLM-Speaker-Diarization-Post-Processing-with-Large-Language.pdf](pdfs/3-asr-y-atribucion/2.03-Wang-2024-DiarizationLM-Speaker-Diarization-Post-Processing-with-Large-Language.pdf)

### 2.4 DiCoW
- Referencia: Alexander Polok, Dominik Klement, Martin Kocour, Jiangyu Han, Federico Landini, Bolaji Yusuf, Matthew Wiesner, Sanjeev Khudanpur et al. (2024). "DiCoW: Diarization-Conditioned Whisper for Target Speaker Automatic Speech Recognition". arXiv. arXiv: https://arxiv.org/abs/2501.00114.
- Qué hace: Whisper condicionado por máscaras de diarización por frame (transformaciones FDDT y sesgo en query-key) para transcribir al hablante objetivo sin depender de embeddings de hablante.
- Relevancia para DASR: Alternativa a la fusión por intervalos: en lugar de asignar las palabras después, se transcribe por hablante.
- Etiquetas: [tiempos por palabra: n/a] [integración: ASR condicionado] [código: https://github.com/BUTSpeechFIT/DiCoW] [relevancia: media]
- PDF: [2.04-Polok-2024-DiCoW-Diarization-Conditioned-Whisper-for-Target-Speaker-Automatic.pdf](pdfs/3-asr-y-atribucion/2.04-Polok-2024-DiCoW-Diarization-Conditioned-Whisper-for-Target-Speaker-Automatic.pdf)

### 2.5 Speech LLM unificado
- Referencia: Phurich Saengthong, Boonnithi Jiaramaneepinit, Sheng Li, Manabu Okumura, Takahiro Shinozaki (2025). "A Unified Speech LLM for Diarization and Speech Recognition in Multilingual Conversations". Workshop del reto MLC-SLM 2025. arXiv: https://arxiv.org/abs/2507.02927.
- Qué hace: Speech LLM que hace diarización y ASR conjuntos en conversaciones multilingües (Task II del reto MLC-SLM). Reformula el formato de entrenamiento para evitar la ambigüedad del audio presegmentado y mejora un 54.87 % relativo en tcpWER/tcpCER sobre el baseline, quedando 8.º.
- Relevancia para DASR: Ejemplo de la línea Speech LLM; sirve como trabajo relacionado y como contraste con la cascada.
- Etiquetas: [tiempos por palabra: n/a] [integración: conjunto] [código: no localizado] [relevancia: media]
- PDF: [2.05-Saengthong-2025-A-Unified-Speech-LLM-for-Diarization-and-Speech.pdf](pdfs/3-asr-y-atribucion/2.05-Saengthong-2025-A-Unified-Speech-LLM-for-Diarization-and-Speech.pdf)

### 2.6 SpeakerLM
- Referencia: Han Yin, Yafeng Chen, Chong Deng, Luyao Cheng, Hui Wang, Chao-Hong Tan, Qian Chen, Wen Wang et al. (2025). "SpeakerLM: End-to-End Versatile Speaker Diarization and Recognition with Multimodal Large Language Models". AAAI 2026. arXiv: https://arxiv.org/abs/2508.06372.
- Qué hace: LLM multimodal que hace diarización y reconocimiento de punta a punta y admite el registro de hablantes conocidos.
- Relevancia para DASR: Une diarización, identificación y ASR en un modelo; contraste con la separación en módulos de DASR.
- Etiquetas: [tiempos por palabra: n/a] [integración: conjunto] [código: no localizado] [relevancia: baja]
- PDF: [2.06-Yin-2025-SpeakerLM-End-to-End-Versatile-Speaker-Diarization-and.pdf](pdfs/3-asr-y-atribucion/2.06-Yin-2025-SpeakerLM-End-to-End-Versatile-Speaker-Diarization-and.pdf)

### 2.7 Sistema de BUT para MLC-SLM
- Referencia: Alexander Polok, Jiangyu Han, Dominik Klement, Samuele Cornell, Jan Černocký, Lukáš Burget (2025). "BUT System for the MLC-SLM Challenge". arXiv. arXiv: https://arxiv.org/abs/2506.13414.
- Qué hace: Combina DiCoW con DiariZen. Sin afinar, DiariZen generaliza mejor que pyannote en escenarios multilingües fuera de dominio; afinados con los datos del reto, ambos mejoran.
- Relevancia para DASR: Muestra que una cascada fuerte (diarización + ASR condicionado) compite con los Speech LLM y que DiariZen supera a pyannote fuera de dominio.
- Etiquetas: [tiempos por palabra: n/a] [integración: cascada] [código: ver DiariZen y DiCoW] [relevancia: media]
- PDF: [2.07-Polok-2025-BUT-System-for-the-MLC-SLM-Challenge.pdf](pdfs/3-asr-y-atribucion/2.07-Polok-2025-BUT-System-for-the-MLC-SLM-Challenge.pdf)

### 2.8 Sistema TEA-ASLP para MLC-SLM
- Referencia: Hongfei Xue, Kaixun Huang, Zhikai Zhou, Shen Huang, Shidong Shang (2025). "The TEA-ASLP System for Multilingual Conversational Speech Recognition and Speech Diarization in MLC-SLM 2025 Challenge". Workshop de Interspeech 2025. arXiv: https://arxiv.org/abs/2507.18051.
- Qué hace: Sistema para las dos tareas del reto MLC-SLM: ASR conversacional multilingüe y diarización con ASR.
- Relevancia para DASR: Otro punto de comparación de la Task II.
- Etiquetas: [tiempos por palabra: n/a] [integración: cascada y conjunto] [código: no localizado] [relevancia: baja]
- PDF: [2.08-Xue-2025-The-TEA-ASLP-System-for-Multilingual-Conversational-Speech.pdf](pdfs/3-asr-y-atribucion/2.08-Xue-2025-The-TEA-ASLP-System-for-Multilingual-Conversational-Speech.pdf)

## 3. Métricas y retos de transcripción multi-hablante

### 3.1 WER, MER y WIL
- Referencia: Andrew Cameron Morris, Viktoria Maier, Phil Green (2004). "From WER and RIL to MER and WIL: improved evaluation measures for connected speech recognition". Interspeech 2004. DOI: https://doi.org/10.21437/interspeech.2004-668.
- Qué hace: Analiza el WER y propone MER y WIL como medidas acotadas de la calidad de una transcripción.
- Relevancia para DASR: Referencia de la métrica WER de la rama ASR; `jiwer` implementa estas medidas.
- Etiquetas: [tiempos por palabra: n/a] [integración: métrica] [código: https://github.com/jitsi/jiwer] [relevancia: media]
- PDF: [3.01-Morris-2004-From-WER-and-RIL-to-MER-and-WIL.pdf](pdfs/3-asr-y-atribucion/3.01-Morris-2004-From-WER-and-RIL-to-MER-and-WIL.pdf)

### 3.2 CHiME-6 y cpWER
- Referencia: Shinji Watanabe, Michael Mandel, Jon Barker, Emmanuel Vincent, Ashish Arora, Xuankai Chang, Sanjeev Khudanpur, Vimal Manohar et al. (2020). "CHiME-6 Challenge:Tackling Multispeaker Speech Recognition for Unsegmented Recordings". Workshop CHiME 2020 (no verificado). arXiv: https://arxiv.org/abs/2004.09249.
- Qué hace: Reto de reconocimiento multi-hablante a distancia en cenas, con una pista sin segmentar que exige diarizar y transcribir; introduce el cpWER (WER concatenado con la mejor permutación de hablantes).
- Relevancia para DASR: El cpWER es la métrica natural para evaluar la salida conjunta de DASR (quién dijo qué).
- Etiquetas: [tiempos por palabra: n/a] [integración: métrica y reto] [código: n/a] [relevancia: media]
- PDF: [3.02-Watanabe-2020-CHiME-6-Challenge-Tackling-Multispeaker-Speech-Recognition-for.pdf](pdfs/3-asr-y-atribucion/3.02-Watanabe-2020-CHiME-6-Challenge-Tackling-Multispeaker-Speech-Recognition-for.pdf)

### 3.3 MeetEval
- Referencia: Thilo von Neumann, Christoph Boeddeker, Marc Delcroix, Reinhold Haeb-Umbach (2023). "MeetEval: A Toolkit for Computation of Word Error Rates for Meeting Transcription Systems". Workshop CHiME-7 2023. arXiv: https://arxiv.org/abs/2307.11394.
- Qué hace: Toolkit de métricas para transcripción de reuniones: cpWER, ORC-WER, MIMO-WER y tcpWER, que añade una restricción temporal para no dar por correctas palabras con tiempos imposibles.
- Relevancia para DASR: Herramienta para medir la salida de `assign_speakers` contra una referencia con hablantes.
- Etiquetas: [tiempos por palabra: los evalúa (tcpWER)] [integración: métrica] [código: https://github.com/fgnt/meeteval] [relevancia: media]
- PDF: [3.03-Neumann-2023-MeetEval-A-Toolkit-for-Computation-of-Word-Error.pdf](pdfs/3-asr-y-atribucion/3.03-Neumann-2023-MeetEval-A-Toolkit-for-Computation-of-Word-Error.pdf)

### 3.4 NOTSOFAR-1
- Referencia: Alon Vinnikov, Amir Ivry, Aviv Hurvitz, Igor Abramovski, Sharon Koubi, Ilya Gurvich, Shai Pe`er, Xiong Xiao et al. (2024). "NOTSOFAR-1 Challenge: New Datasets, Baseline, and Tasks for Distant Meeting Transcription". arXiv (reto CHiME-8). arXiv: https://arxiv.org/abs/2401.08887.
- Qué hace: Reto de transcripción de reuniones con micrófono lejano, con datos reales y simulados y evaluación con tcpWER.
- Relevancia para DASR: Muestra el estado de la transcripción multi-hablante en condiciones difíciles.
- Etiquetas: [tiempos por palabra: n/a] [integración: reto] [código: n/a] [relevancia: baja]
- PDF: [3.04-Vinnikov-2024-NOTSOFAR-1-Challenge-New-Datasets-Baseline-and-Tasks.pdf](pdfs/3-asr-y-atribucion/3.04-Vinnikov-2024-NOTSOFAR-1-Challenge-New-Datasets-Baseline-and-Tasks.pdf)

## Respuesta directa

La forma estándar de obtener tiempos por palabra es la de WhisperX, que es la que adopta DASR: Whisper transcribe y un modelo wav2vec 2.0 con CTC alinea cada palabra. La unión con la diarización se hace por intersección temporal. La investigación reciente explora modelos conjuntos (DiCoW, Speech LLM, SpeakerLM) y post-procesado con LLM (DiarizationLM), pero los resultados de 2025 y 2026 (el sistema de BUT en MLC-SLM, Amorim et al.) indican que una cascada bien hecha no está en desventaja.

## Huecos detectados

1. No se encontró un estudio que mida cuánto error de atribución de palabras viene de las fronteras entre turnos en una cascada tipo WhisperX, separado del error de diarización.
2. La alineación forzada en español con los modelos que usa WhisperX no está evaluada en los trabajos revisados.
3. No hay comparación directa entre la diarización con solapamiento y la diarización exclusiva de pyannote 4.0 para la fusión con el ASR.
