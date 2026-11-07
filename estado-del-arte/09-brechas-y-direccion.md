# 9. Brechas abiertas y dirección del proyecto

## 9.1 Problemas abiertos del área

1. **Muchos hablantes y solapamiento.** Todos los sistemas empeoran cuando hay más de 4–5 hablantes o mucha habla simultánea. Sortformer está limitado a 4; Nemotron 3 sube a 8.
2. **Habla perdida.** Según el benchmark de 2025, la principal fuente de error es la voz no detectada, por encima de la confusión de hablantes.
3. **Desajuste de dominio e idioma.** Casi todo se entrena y evalúa en inglés y chino. Hay pocos datos anotados en español (RTVE es la excepción, y pide licencia).
4. **Alinear palabras con hablantes.** En cascada, las fronteras del ASR y de la diarización no coinciden; los modelos conjuntos (DiCoW, Speech LLMs) intentan resolverlo.
5. **Identificación en conjunto abierto.** Decidir "desconocido" con un umbral robusto entre canales e idiomas sigue siendo delicado.
6. **Valor real del video.** MISP 2025 muestra que el video aporta poco a la diarización en reuniones; su valor está en el "dónde" visual y en los casos difíciles.
7. **Costo computacional.** Los mejores modelos (WavLM-Large, Speech LLMs) son pesados; podarlos (DiariZen) o hacerlos ligeros es un área activa.

## 9.2 Propuesta de pipeline para el proyecto

Una arquitectura modular que responde las cuatro preguntas y permite medir cada parte por separado:

```
             ┌──────────────────────────────────────────────────────────┐
video/audio ─┤ ffmpeg → audio 16 kHz mono                               │
             └──────────────────────────────────────────────────────────┘
                     │
        ┌────────────┼──────────────────────┐
        ▼            ▼                      ▼
 Diarización     ASR (Whisper)        [solo video] caras + ASD
 (pyannote c-1   palabras + tiempos   (TalkNet / Light-ASD)
  o DiariZen)          │                      │
        │              │                      │
        ▼              ▼                      ▼
 Embeddings por   Alineación            Asociar SPK_k ↔ cara
 hablante         palabra → hablante    (solapamiento temporal)
        │              │                      │
        ▼              │                      │
 Identificación   ─────┴──────────────────────┘
 (coseno contra voces registradas,
  umbral → nombre o "desconocido")
                     │
                     ▼
     Salida: [inicio, fin, nombre, texto, cara/posición]
```

| Pregunta | Módulo | Modelo base | Métrica |
|----------|--------|-------------|---------|
| ¿Cuándo? | Diarización | pyannote *community-1* / DiariZen | DER, JER |
| ¿Quién? | Identificación | ECAPA-TDNN / ResNet + coseno | Exactitud en conjunto abierto, EER |
| ¿Qué? | ASR + alineación | Whisper / WhisperX | WER, WDER, cpWER |
| ¿Dónde? | Timestamps + ASD | TalkNet / Light-ASD | mAP o exactitud de asociación voz-cara |

## 9.3 Posibles aportes propios (lo que hace el proyecto "de ML")

Usar modelos preentrenados está bien como base, pero el proyecto gana valor si incluye al menos un experimento propio. Opciones, de menos a más trabajo:

1. **Comparativa rigurosa en español:** pyannote vs. DiariZen vs. Nemotron 3 sobre un conjunto propio, con DER a collar 0 y 0.25 y análisis de errores (habla perdida vs. confusión, por número de hablantes).
2. **Identificación abierta calibrada:** comparar embeddings (ECAPA, ResNet, CAM++, WavLM) y estrategias de umbral para "desconocido" con voces en español; reportar EER y exactitud.
3. ***Fine-tuning* del segmentador** con datos en español siguiendo la receta del paper de pyannote 2.1 (el link que pasaste). Hubo equipos que lo hicieron en Albayzin 2024.
4. **Cascada vs. condicionado:** WhisperX contra DiCoW en cpWER/WDER.
5. **Post-procesado con LLM** al estilo DiarizationLM sobre la salida de la cascada, con un LLM abierto pequeño.
6. **Extensión audiovisual:** añadir ASD y medir si ayuda a corregir la diarización en videos con caras visibles.

## 9.4 Decisiones a tomar en equipo

- ¿Qué significa "dónde" en el proyecto: solo marcas de tiempo, o también la cara en el video?
- ¿Idioma principal de evaluación: español, inglés o ambos?
- ¿Identificación con personas **conocidas** (hace falta un registro de voces) o solo diarización anónima?
- ¿Hay GPU disponible? Eso decide entre pyannote (corre en CPU) y DiariZen/Nemotron (mejor con GPU).
