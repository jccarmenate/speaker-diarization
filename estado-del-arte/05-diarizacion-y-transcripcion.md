# 5. Diarización + transcripción: "quién dijo qué"

Para saber **qué** dijo cada hablante hay que combinar diarización con ASR. Hay tres generaciones de enfoques.

## 5.1 Cascada: ASR + diarización + alineación

```
audio ─┬─ ASR (Whisper) → palabras con tiempos ──┐
       └─ diarización → segmentos por hablante ──┴→ asignar hablante a cada palabra
```

- **Whisper** ([Radford et al., 2022](https://arxiv.org/abs/2212.04356)): ASR multilingüe robusto, muy bueno en español. No hace diarización y sus marcas de tiempo por palabra son imprecisas.
- **WhisperX** ([Bain et al., 2023](https://arxiv.org/abs/2303.00747)): Whisper (vía faster-whisper) + VAD + **alineación forzada con wav2vec2** para tiempos precisos por palabra + diarización con pyannote. Asigna a cada palabra el hablante con el que más se superpone. **Es el baseline práctico más usado.**
- **Problema:** las fronteras del ASR y de la diarización no coinciden; las palabras en un cambio de turno o en un solapamiento se asignan mal. La *diarización exclusiva* de pyannote *community-1* existe justamente para facilitar esta alineación.

## 5.2 Post-procesado con LLM

- **DiarizationLM** ([Wang et al., Google, 2024](https://arxiv.org/abs/2401.03506)): convierte la salida de ASR + diarización en texto compacto y le pide a un LLM afinado (PaLM 2-S) que corrija las etiquetas de hablante. Reduce la tasa de error de diarización por palabra (**WDER**) un 55.5% relativo en Fisher y entre ~31% y ~45% en CALLHOME inglés (el número varía según la versión del paper). Se aplica sobre cualquier sistema sin reentrenarlo.
- La idea: el LLM usa el **contenido** ("¿y tú qué opinas?" anuncia un cambio de turno) que el modelo acústico no ve.

## 5.3 Modelos conjuntos (2024–2026)

| Modelo | Año | Idea |
|--------|-----|------|
| **Sortformer + ASR** ([Park et al.](https://arxiv.org/abs/2409.06656)) | 2024 | El orden por llegada permite inyectar la salida de diarización en un ASR multi-hablante sin resolver permutaciones. |
| **DiCoW** ([Polok et al., BUT](https://arxiv.org/abs/2501.00114)) | dic. 2024 | *Diarization-Conditioned Whisper*: Whisper recibe máscaras de diarización por frame y transcribe solo al hablante objetivo. No necesita modelar identidades. Con DiariZen quedó **2.º en MLC-SLM 2025**. |
| **Unified Speech LLM** ([Saengthong et al.](https://arxiv.org/abs/2507.02927)) — **el link que pasaste** | jun. 2025 | Un Speech LLM (con un LLM pequeño) que hace diarización + ASR de punta a punta. Cambia el formato de entrenamiento para evitar la ambigüedad del audio pre-segmentado. 54.87% de mejora relativa en tcpWER/tcpCER sobre el baseline; 8.º en la Task II. |
| **SpeakerLM** ([arXiv 2508.06372](https://arxiv.org/abs/2508.06372)) | ago. 2025 | LLM multimodal (Qwen2.5-7B) que hace diarización + reconocimiento de punta a punta y admite **registro de hablantes conocidos**: une diarización, identificación y ASR en un modelo. |
| Sistemas MLC-SLM ([TEA-ASLP](https://arxiv.org/abs/2507.18051), [BUT](https://arxiv.org/abs/2506.13414), DKU…) | 2025 | Las mejores entradas combinan un buen diarizador (DiariZen, pyannote) con un ASR condicionado o un decodificador LLM. |

### El reto MLC-SLM 2025
*Multilingual Conversational Speech Language Model Challenge* (Interspeech 2025). Usa conversaciones reales en ~11 idiomas (*verificar si incluye español*) y tiene dos tareas: ASR con segmentación de referencia (Task I) y **diarización + ASR sin información de referencia** (Task II), evaluada con **tcpWER/tcpCER**. Es el benchmark más directo para "quién dijo qué".

## 5.4 Lectura del estado del arte

- **Cascada (WhisperX)**: la más fácil de montar y suficiente como baseline.
- **Condicionada (DiCoW)**: hoy lo más competitivo en retos, sin depender de un LLM gigante.
- **Speech LLM (2507.02927, SpeakerLM)**: la dirección de investigación más activa; caro de entrenar, pero útil como trabajo relacionado y para comparar.

Para un proyecto de curso, lo razonable es **implementar la cascada** y **discutir** los modelos conjuntos como estado del arte, o probar DiCoW si hay GPU.
