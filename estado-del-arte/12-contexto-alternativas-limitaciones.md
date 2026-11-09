# 12. Contexto, alternativas y limitaciones

Este archivo sitúa la propuesta dentro del panorama actual (noviembre de 2026): qué otras herramientas hay, cómo se comparan y qué problemas siguen abiertos.

## 12.1 Las tres herramientas de referencia

| | **pyannote.audio** | **WhisperX** | **NVIDIA NeMo** |
|---|--------------------|--------------|-----------------|
| **Qué es** | Toolkit de diarización | Pipeline de ASR + alineación + diarización | Toolkit general de voz (ASR, diarización, TTS) |
| **Diarización** | Propia: segmentación + embeddings + clustering (3.1: AHC; 4.0: VBx) | Usa pyannote | Propia: MSDD, Sortformer, Nemotron 3 |
| **ASR** | No | Whisper + wav2vec 2.0 (alineación) | Canary, Parakeet |
| **Código** | [GitHub](https://github.com/pyannote/pyannote-audio), MIT | [GitHub](https://github.com/m-bain/whisperX) | [GitHub](https://github.com/NVIDIA/NeMo), Apache 2.0 |
| **Papel en DASR** | **Referencia que reproducimos** | **Referencia de la rama ASR** | Alternativa para comparar |

### NeMo en detalle

NeMo sigue dos líneas de diarización:

1. **Pipeline en cascada con decodificador multiescala:**
   - **TitaNet** ([Koluguri et al., 2021](https://arxiv.org/abs/2110.04410)): modelo de embeddings de hablante.
   - **MSDD** ([Park et al., 2022](https://arxiv.org/abs/2203.15974), *Multi-scale Diarization Decoder*): extrae embeddings a **varias escalas temporales** (segmentos largos y cortos) y aprende a ponderarlas. Los segmentos largos dan identidad fiable; los cortos, resolución temporal. Es otra forma de resolver el mismo dilema local/global que pyannote.
2. **End-to-end con Sortformer:**
   - **Sortformer** ([Park et al., 2024](https://arxiv.org/abs/2409.06656)): una red predice directamente la actividad de cada hablante. Resuelve el problema de permutación **ordenando a los hablantes por orden de llegada** en lugar de con clustering. Máximo 4 hablantes.
   - **Streaming Sortformer** ([2025](https://arxiv.org/abs/2507.18446)): versión en tiempo real con una caché de hablantes.
   - **Nemotron 3 Diarization** ([septiembre 2026](https://huggingface.co/nvidia/Nemotron-3-Diarization)): hasta 8 hablantes, modo offline y en tiempo real.

**Por qué no elegimos NeMo como base:** los modelos end-to-end son más difíciles de explicar paso a paso (el problema de permutación se resuelve dentro de la red, no con un algoritmo visible), el límite de hablantes es fijo y NeMo es un toolkit grande y pesado para reproducir a mano. Sigue siendo una buena **comparación externa**.

## 12.2 Otros sistemas relevantes

| Sistema | Idea | Relación con DASR |
|---------|------|-------------------|
| **DiariZen** (BUT, [2024](https://arxiv.org/abs/2409.09408)–[2026](https://arxiv.org/abs/2604.21507)) | Misma estructura que pyannote, con WavLM afinado en la segmentación y VBx en el clustering | **Evolución directa** de lo que reproducimos; tiene un tutorial de 2026 con notebook |
| **EEND** ([Fujita et al., 2019](https://arxiv.org/abs/1909.06247)) y variantes | Diarización completamente end-to-end | Origen de la segmentación neuronal de pyannote |
| **EEND-VC** ([Kinoshita et al., 2020](https://arxiv.org/abs/2010.13366)) | EEND por bloques + clustering de vectores entre bloques | Es la misma idea "local + global" de pyannote |

### Benchmark independiente reciente

[*Benchmarking Diarization Models*](https://arxiv.org/abs/2509.26177) (ETH Zurich, septiembre 2025) evaluó cinco sistemas sobre 196.6 horas en cinco idiomas, **incluido el español**:
- mejor sistema comercial: pyannoteAI, **11.2 %** de DER;
- mejor abierto: DiariZen, **13.3 %**;
- Sortformer, muy fuerte con pocos hablantes, empeora cuando hay más de 4.

## 12.3 Modelos conjuntos diarización + ASR

DASR une ASR y diarización **después** de calcularlas por separado (cascada). La investigación reciente intenta hacerlo **en un solo modelo**:

| Enfoque | Ejemplo | Idea |
|---------|---------|------|
| ASR condicionado por diarización | **DiCoW** ([Polok et al., 2024](https://arxiv.org/abs/2501.00114)) | Whisper recibe la diarización como entrada y transcribe solo al hablante objetivo |
| *Speech LLM* unificado | [Saengthong et al., 2025](https://arxiv.org/abs/2507.02927) | Un modelo de lenguaje con entrada de audio genera la transcripción con etiquetas de hablante. Mejora 54.87 % relativo sobre el baseline del reto MLC-SLM |
| LLM multimodal | **SpeakerLM** ([2025](https://arxiv.org/abs/2508.06372)) | Diarización + reconocimiento en un LLM; admite registro de hablantes conocidos |
| Post-procesado con LLM | **DiarizationLM** ([Wang et al., 2024](https://arxiv.org/abs/2401.03506)) | Un LLM corrige las etiquetas de hablante de la salida en cascada usando el contenido del texto |

Son la dirección más activa del área, pero caros de entrenar y difíciles de explicar paso a paso. Para el proyecto sirven como **trabajo relacionado** y como posibles extensiones (DiarizationLM, por ejemplo, se puede aplicar sobre la salida de DASR sin reentrenar nada).

## 12.4 Limitaciones actuales de la diarización

Problemas que siguen abiertos incluso en los mejores sistemas, y cómo afectan a DASR:

| Limitación | Descripción | En DASR |
|------------|-------------|---------|
| **Habla perdida** | El benchmark de 2025 encontró que la principal fuente de error es la voz no detectada, por encima de la confusión de hablantes. | Depende de la Fase 1: el fine-tuning de la segmentación es la palanca principal. |
| **Muchos hablantes** | Todos los sistemas empeoran con más de 4–5 hablantes. | La segmentación de pyannote admite 3 por ventana (10 s), pero el clustering no tiene límite global. |
| **Solapamiento** | Sigue siendo la parte más difícil. | Powerset admite 2 a la vez; un tercero simultáneo se pierde. |
| **Turnos muy cortos** | "Sí", "ajá": poca voz para un embedding fiable. | Mitigado por la máscara de ventana completa (Fase 2), no resuelto. |
| **Desajuste de dominio e idioma** | Los modelos se entrenan sobre todo en inglés y chino. | Evaluar con audio en español y, si es necesario, afinar. |
| **Elección del umbral** | `δ` fijo puede no servir para todos los audios. | Optimizar `δ` con datos propios ([archivo 11](11-fine-tuning.md)). |
| **Fronteras ASR ↔ diarización** | Las palabras en cambios de turno se asignan mal. | Usar la diarización exclusiva en la fusión ([archivo 7](07-integracion-asr-diarizacion.md)). |
| **Atributos vocales** | Pocos resultados sólidos fuera de modelos entrenados específicamente; riesgo de sesgo. | Reportar probabilidades y evaluar por subgrupos ([archivo 8](08-atributos-vocales.md)). |

## 12.5 Por qué la propuesta sigue siendo la adecuada

- La arquitectura **local + global** de pyannote es la base de los mejores sistemas abiertos actuales (pyannote 4.0, DiariZen): las mejoras recientes **cambian piezas** (WavLM, VBx), no la estructura.
- Cada pieza es **explicable** con contenidos del curso: redes recurrentes, embeddings, clustering jerárquico, métricas.
- Al reproducirla módulo a módulo, las mejoras modernas quedan como **extensiones naturales**: cambiar AHC por VBx, cambiar la segmentación por WavLM, añadir post-procesado con LLM.
