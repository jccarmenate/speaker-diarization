# 7. Datasets y métricas

## 7.1 Métricas

### Diarización
- **DER** (*Diarization Error Rate*): la métrica principal.

  `DER = (falsa alarma + habla perdida + confusión de hablante) / duración total de habla`

  Al comparar cifras hay que mirar **cómo se calculó**:
  - **collar**: margen sin evaluar alrededor de cada frontera (0 s o 0.25 s). Con *collar* el DER baja bastante;
  - si se **cuenta el solapamiento** o se ignora;
  - si se usó **VAD de referencia** (*oracle*) o automático.

  Las cifras modernas serias (pyannote, ETH 2025) usan **collar 0, con solapamiento y VAD automático**: la configuración más estricta.
- **JER** (*Jaccard Error Rate*): como DER pero pondera a todos los hablantes por igual (en DER, los que hablan poco casi no cuentan). Se usó en DIHARD.
- Herramientas para calcularlas: `pyannote.metrics`, `dscore` (DIHARD), `md-eval`.

### Diarización + transcripción
- **WDER** (*Word Diarization Error Rate*): fracción de palabras con hablante incorrecto.
- **cpWER** (*concatenated minimum-permutation WER*): se concatena el texto de cada hablante y se calcula el WER con la mejor asignación de hablantes. Introducido en CHiME-6.
- **tcpWER** (*time-constrained cpWER*): como cpWER pero exige que las palabras caigan cerca de su tiempo real. Lo usan CHiME-8, NOTSOFAR-1 y MLC-SLM. Herramienta: `meeteval`.

### Identificación / verificación
- **EER** (*Equal Error Rate*): punto en que falsa aceptación = falso rechazo.
- **minDCF**: costo de detección mínimo con pesos dados.
- **Exactitud top-1** (conjunto cerrado) y **tasa de detección e identificación a una tasa de falsas alarmas fija** (conjunto abierto).

### Audiovisual
- **mAP** en AVA-ActiveSpeaker para ASD; DER para diarización audiovisual.

## 7.2 Datasets de diarización

| Dataset | Dominio | Idioma | Notas |
|---------|---------|--------|-------|
| **AMI** | Reuniones (100 h) | Inglés | Clásico; varios micrófonos + video. |
| **CALLHOME** (NIST SRE 2000) | Teléfono, 2–7 hablantes | Multilingüe (**incluye español**) | Muy usado para EEND; de pago (LDC). |
| **DIHARD III** ([Ryant et al.](https://arxiv.org/abs/2012.01477)) | 11 dominios difíciles | Inglés (sobre todo) | El benchmark "duro"; de pago (LDC). |
| **VoxConverse** ([Chung et al.](https://arxiv.org/abs/2007.01216)) | Debates y noticias de YouTube | Inglés (sobre todo) | **Libre**; audio + video. Bueno para el proyecto. |
| **AISHELL-4** | Reuniones (120 h, 8 micrófonos) | Mandarín | Libre. |
| **AliMeeting / M2MeT** ([Yu et al.](https://arxiv.org/abs/2110.07393)) | Reuniones | Mandarín | Libre. |
| **MSDWild** | Videos "en la naturaleza" | Multilingüe | Libre; audio + video. |
| **Ego4D** | Video en primera persona | Inglés y otros | Requiere licencia. |
| **NOTSOFAR-1** ([Vinnikov et al.](https://arxiv.org/abs/2401.08887)) | Reuniones con micrófono lejano | Inglés | Reto CHiME-8. |
| **CHiME-6/7/8** | Cenas, reuniones | Inglés | Retos de transcripción multi-hablante. |
| **MISP 2025** ([arXiv 2505.13971](https://arxiv.org/abs/2505.13971)) | Reuniones audio + video (125 h) | Mandarín | Diarización audiovisual. |
| **AVA-AVD** ([Xu et al.](https://arxiv.org/abs/2111.14448)) | Películas | Multilingüe | Diarización audiovisual. |
| **MLC-SLM 2025** | Conversaciones | ~11 idiomas | Diarización + ASR; *verificar si incluye español*. |

### En español
| Dataset | Notas |
|---------|-------|
| **RTVE / Albayzin** ([Cátedra RTVE – Univ. de Zaragoza](https://catedrartve.unizar.es/rtvedatabase.html)) | Programas de televisión española. Hay retos de diarización e identidad en 2018, 2020, 2022 y 2024 (*Speaker Diarization and Identity Assignment Challenge*, IberSPEECH 2024). RTVE2018: ~569 h. Gratuito para investigación **con solicitud de licencia** del grupo. **Es el más relevante si el proyecto trabaja en español.** |
| **CALLHOME Spanish** | Parte en español de CALLHOME (LDC, de pago). |
| **Benchmark ETH 2025** ([arXiv 2509.26177](https://arxiv.org/abs/2509.26177)) | Incluye audio en español entre sus 196.6 h; útil como referencia de cifras. |

## 7.3 Datasets de identificación de hablantes
- **VoxCeleb1 / VoxCeleb2**: >7 000 famosos, videos de YouTube; estándar para entrenar y evaluar embeddings. Libre para investigación.
- **CN-Celeb**: chino, multigénero.
- **VoxBlink / VoxBlink2**: escala mucho mayor, extraído de YouTube.

## 7.4 Recomendación para el proyecto
- **Evaluación principal:** VoxConverse (libre, inglés, con video) + un conjunto pequeño **en español** anotado por el equipo (por ejemplo 30–60 min de un podcast o una entrevista) o RTVE si se consigue la licencia.
- **Métricas a reportar:** DER (collar 0 y 0.25, con solapamiento), JER, WDER/cpWER si se transcribe, y exactitud de identificación en conjunto abierto.
