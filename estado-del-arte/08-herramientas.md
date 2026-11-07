# 8. Herramientas y modelos disponibles

Panorama de toolkits abiertos a noviembre de 2026. Revisar siempre la licencia del **modelo** (no solo la del código): varios checkpoints restringen el uso comercial o piden aceptar condiciones en Hugging Face.

## 8.1 Diarización

| Herramienta | Qué ofrece | Notas |
|-------------|-----------|-------|
| **pyannote.audio** ([GitHub](https://github.com/pyannote/pyannote-audio)) | Pipeline completo; modelo abierto actual `pyannote/speaker-diarization-community-1` (pyannote.audio 4.0) | Código MIT; modelo CC-BY-4.0, hay que aceptar condiciones en Hugging Face. API muy simple; receta de *fine-tuning* (el paper de 2.1 que pasaste). Versión comercial: pyannoteAI *Precision-2*. |
| **DiariZen** ([GitHub BUTSpeechFIT](https://github.com/BUTSpeechFIT/DiariZen)) | WavLM podado + Conformer + VBx; el mejor abierto en el benchmark de ETH 2025 | Tutorial con notebook: [arXiv 2604.21507](https://arxiv.org/abs/2604.21507). Revisar la licencia de cada modelo. Más pesado que pyannote. |
| **NVIDIA NeMo** ([GitHub](https://github.com/NVIDIA/NeMo)) | Sortformer, Streaming Sortformer, **Nemotron 3 Diarization**, MSDD, TitaNet | Apache 2.0 (código). Ideal para streaming; necesita GPU NVIDIA para ir cómodo. |
| **SpeechBrain** | ECAPA-TDNN preentrenado, recetas de diarización con clustering espectral | Apache 2.0. Muy didáctico. |
| **WeSpeaker** | Embeddings ResNet/CAM++/ECAPA + diarización basada en clustering | Los embeddings de pyannote 3.x salen de aquí. |
| **3D-Speaker** (Alibaba) | CAM++, ERes2Net, diarización, incluso multimodal | Fuerte en chino. |

## 8.2 Transcripción con hablantes

| Herramienta | Qué ofrece |
|-------------|-----------|
| **Whisper / faster-whisper** | ASR multilingüe (muy bueno en español). faster-whisper es una reimplementación más rápida con CTranslate2. |
| **WhisperX** ([GitHub](https://github.com/m-bain/whisperX)) | Whisper + alineación por palabra (wav2vec2) + diarización pyannote. Baseline de "quién dijo qué" casi listo. |
| **DiCoW** (BUT) | Whisper condicionado por diarización; código y modelos de BUT. |
| **NeMo ASR (Canary / Parakeet)** | ASR de NVIDIA, integrable con Sortformer. |

## 8.3 Video y hablante activo

| Herramienta | Qué ofrece |
|-------------|-----------|
| **TalkNet-ASD** | Código y modelo preentrenado en AVA-ActiveSpeaker; incluye demo sobre videos propios. |
| **Light-ASD** | ASD ligero (CVPR 2023). |
| **Detección y tracking de caras** | RetinaFace / S3FD + tracking (por ejemplo ByteTrack) para obtener las pistas de cara que consume el ASD. |
| **ffmpeg** | Extraer el audio del video (`ffmpeg -i video.mp4 -ac 1 -ar 16000 audio.wav`). Casi todos los modelos esperan **16 kHz mono**. |

## 8.4 Evaluación

| Herramienta | Métricas |
|-------------|----------|
| `pyannote.metrics` | DER, JER, pureza/cobertura, con o sin collar |
| `dscore` | DER/JER oficiales de DIHARD |
| `meeteval` | cpWER, tcpWER (CHiME-8, MLC-SLM) |

## 8.5 Combinación recomendada para empezar

- **Rápido y bien documentado:** pyannote *community-1* + faster-whisper/WhisperX + embeddings ECAPA (SpeechBrain) para identificar.
- **Máxima precisión offline:** DiariZen en lugar de pyannote.
- **Tiempo real o más de 4 hablantes:** Nemotron 3 Diarization (NeMo).
