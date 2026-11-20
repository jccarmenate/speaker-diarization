# 1. Visión general y arquitectura

## 1.1 Hoja de ruta

El marco teórico se recorre en cinco bloques. Cada uno responde una pregunta:

| # | Bloque | Pregunta | Contenido |
|---|--------|----------|-----------|
| 01 | Visión general | ¿Qué queremos y cómo se organiza? | Arquitectura conceptual y del código |
| 02 | Fundamentos | ¿Qué es el ASR y qué es la diarización? | Por qué el enfoque de pyannote |
| 03 | Diarización en 3 fases | ¿Quién habló y cuándo? | Preparación → procesamiento local → procesamiento global |
| 04 | Integración y atributos | ¿Quién dijo qué y cómo es su voz? | Unir palabras con hablantes; analizar la voz |
| 05 | Implementación | ¿Cómo lo construimos y lo medimos? | Mapa del código, métricas y plan |

El núcleo es el bloque 03: la diarización en tres fases tal como la plantea pyannote.

## 1.2 Arquitectura conceptual

```
                       ┌──────────────── ASR: ¿qué se dijo? ────────────────┐
                       │  transcripción + palabras con marcas de tiempo      │
                       │                                                     ▼
   AUDIO ──────────────┤                                       Fusión palabra ↔ hablante ──► SALIDA FINAL
                       │                                       (transcripción por hablante)   quién dijo qué,
                       │                                                     ▲                 cuándo y cómo
                       └──── DIARIZACIÓN: ¿quién habló y cuándo? ────────────┤                 es su voz
                             Fase 1 · Fase 2 · Fase 3                        │                     ▲
                             segmentos temporales por hablante ──────────────┘                     │
                             embeddings de voz (huella vocal) ──► Análisis de patrones ──► Atributos vocales
                                                                  (rasgos + modelos)       (sexo, edad, acento…)
```

**Cómo leerla:**

1. El audio entra a **dos ramas independientes**.
2. La rama de **ASR** responde "¿qué se dijo?" y produce palabras, cada una con su instante de inicio y fin.
3. La rama de **diarización** responde "¿quién habló cuándo?" y produce segmentos temporales con una etiqueta de hablante.
4. Como **ambas salidas comparten el eje del tiempo**, se pueden fusionar: cada palabra recibe el hablante cuyo segmento la contiene.
5. Por cada hablante tenemos además **embeddings** (huellas vocales) y fragmentos de audio. Con ellos hacemos el **análisis de patrones** que lleva a los **atributos vocales**. Ese análisis se hace **por hablante**, no por palabra.

> **Idea clave:** dos ramas independientes (texto y hablantes) que se unen por el tiempo; el análisis vocal se hace por hablante.

Que las ramas sean independientes tiene dos ventajas prácticas: cada una se puede evaluar por separado (WER para el ASR, DER para la diarización) y cada una se puede mejorar o reemplazar sin tocar la otra.

## 1.3 Arquitectura del código: proyecto DASR

```
DASR/
├── pyproject.toml
├── tests/test_wiring.py
└── src/
    ├── Core/            tipos compartidos y carga de audio
    ├── Segmentation/    Fase 1 (segmentación local + binaria)
    ├── ASR/
    │   ├── VAD/         detección de voz
    │   └── FA/          alineación forzada
    ├── D/               diarización
    │   ├── LSE/         Local Speaker Embedding (Fase 2)
    │   └── GAC/         Global Agglomerative Clustering (Fase 3)
    ├── Pipeline/        fusión palabra ↔ hablante
    ├── Visualization/   figuras de cada etapa
    └── AVA/             Análisis de atributos vocales
```

### Flujo de datos entre módulos

```
load_audio ──► AudioSignal (mono, 16 kHz)
                    │
                    ▼
        Segmentation.run ──► SegmentationOutput
                             (local_segments + binary_segments)
                    │                         │
         (la misma segmentación alimenta a ambos)
                    ▼                         ▼
   ASR.run: VAD → transcribe → FA      D.run: LSE (Fase 2) → GAC (Fase 3) → build_turns
                    │                         │
                    ▼                         ▼
     ASROutput (palabras + tiempos)    DiarizationOutput (turnos por hablante global)
                    └──────────┬──────────────┘
                               ▼
              Pipeline.assign_speakers ──► PipelineOutput ──► AVA
              (transcripción por hablante)                    (atributos vocales potenciales)
```

| Módulo | Responsabilidad |
|--------|-----------------|
| **Core** | Define los tipos de datos (`AudioSignal`) y carga el audio, convertido a **mono y 16 kHz**, el formato que esperan casi todos los modelos de voz. |
| **Segmentation** | Implementa la **Fase 1**: produce la segmentación local (probabilidades por ventana) y la segmentación binaria. |
| **ASR** | Usa la segmentación como **VAD**, transcribe con Whisper y hace **alineación forzada (FA)** para obtener tiempos precisos por palabra. |
| **D** | Diarización: extrae embeddings locales (**LSE**, Fase 2), los agrupa globalmente (**GAC**, Fase 3) y construye los turnos de habla (`build_turns`). |
| **Pipeline** | `assign_speakers` une las salidas del ASR y de la diarización. |
| **AVA** | Análisis de atributos vocales a partir de los hablantes ya identificados. |
| **Visualization** | Genera las figuras de cada etapa: sirven para depurar y para explicar. |

> **Idea clave:** la segmentación se calcula **una sola vez** y alimenta a dos consumidores: al ASR (como detector de voz) y a la diarización (como base de los embeddings).

Esto evita cargar un segundo modelo de VAD y garantiza que ASR y diarización "vean" la misma voz: si la segmentación dice que en un instante no habla nadie, ninguna de las dos ramas producirá nada ahí.
