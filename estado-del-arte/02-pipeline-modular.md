# 2. Pipeline modular (en cascada)

Es el enfoque clásico y sigue siendo la base de los sistemas híbridos actuales.

```
audio → VAD → segmentación → embedding por segmento → clustering → (resegmentación) → etiquetas
```

## Etapas

### 2.1 Detección de voz (VAD)
Separa voz de silencio, música y ruido. Hoy se usan VAD neuronales (por ejemplo Silero VAD o el VAD integrado en pyannote). Los errores aquí se arrastran a todo lo demás: el benchmark de 2025 ([Benchmarking Diarization Models](https://arxiv.org/abs/2509.26177)) encuentra que **la causa principal de error es habla no detectada** (*missed speech*), seguida de confusión de hablantes.

### 2.2 Segmentación
Corta la voz en trozos cortos, en principio de un solo hablante: ventanas fijas (1–2 s con solape) o detección de cambio de hablante. El enfoque moderno reemplaza esta etapa por un **segmentador neuronal** que también detecta solapamiento ([subtema 3](03-end-to-end.md)).

### 2.3 Embeddings de hablante
Cada segmento se convierte en un vector que representa la voz (x-vector, ECAPA-TDNN, ResNet…). Detalle en el [subtema 4](04-embeddings-e-identificacion.md).

### 2.4 Clustering
Agrupa los embeddings por hablante sin saber cuántos hay:

| Método | Idea | Notas |
|--------|------|-------|
| **AHC** (aglomerativo jerárquico) | Une los dos grupos más parecidos hasta un umbral | Simple; el umbral fija el número de hablantes. Lo usa pyannote. |
| **Espectral** | Grafo de afinidad + autovectores | Estima bien el número de hablantes (variante NME-SC de Park et al.). Lo usa NeMo. |
| **VBx** | HMM bayesiano sobre secuencias de x-vectors ([Landini et al., 2020](https://arxiv.org/abs/2012.14952)) | Muy robusto; es el clustering de DiariZen y referencia en DIHARD. |

### 2.5 Resegmentación y solapamiento
El pipeline puramente modular asume **un hablante a la vez**, y ese es su punto débil: en reuniones el solapamiento puede ser 10–20% del habla. Las soluciones fueron añadir un detector de solapamiento, resegmentar con VB-HMM o pasar al enfoque híbrido.

## pyannote: la evolución del pipeline de referencia

| Versión | Año | Cambio principal |
|---------|-----|------------------|
| pyannote.audio 1.x | 2019 | Bloques neuronales por etapa ([Bredin et al.](https://arxiv.org/abs/1911.01255)) |
| **pyannote.audio 2.1** | 2022–2023 | Pipeline de 3 etapas: segmentación local en ventana deslizante (hasta 3 hablantes por ventana, con solapamiento) → embedding de cada hablante local → AHC global. **Es el paper que pasaste** ([Bredin, Interspeech 2023](https://www.isca-archive.org/interspeech_2023/bredin23_interspeech.html)). |
| pyannote 3.0 / 3.1 | 2023 | Segmentador entrenado con pérdida **powerset** ([Plaquet & Bredin, 2023](https://arxiv.org/abs/2310.13025)) y embeddings ResNet34 de WeSpeaker. La 3.1 es PyTorch puro (sin onnxruntime). |
| **community-1** (pyannote.audio 4.0) | 2025 | Modelo abierto actual (licencia CC-BY-4.0). Añade **diarización exclusiva**: una salida sin solapamientos que facilita alinear con las palabras del ASR. |

La idea central del paper de 2.1 sigue vigente: **resolver localmente** lo difícil (solapamiento, cambios rápidos) con una red en ventanas cortas, y **resolver globalmente** la identidad con clustering.

Según la ficha del modelo, *community-1* mejora a 3.1 en todos los datasets académicos que reporta (por ejemplo AliMeeting canal 1: 24.5% → 20.3% de DER; DIHARD III: 21.4% → 20.2%), evaluado sin *collar* y contando el solapamiento. *Estas cifras las tomé de una copia de la ficha; la oficial pide iniciar sesión en Hugging Face. Verificarlas antes de citarlas.*

## Ventajas y límites

- ✅ Interpretable y modular: se puede cambiar cada pieza (otro VAD, otros embeddings).
- ✅ No limita el número de hablantes.
- ✅ Los embeddings sirven directamente para la **identificación** (comparar con voces registradas).
- ❌ Errores en cascada: lo que pierde el VAD no se recupera.
- ❌ El clustering clásico no modela el solapamiento (de ahí el paso a híbridos).
