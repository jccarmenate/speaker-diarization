# 2. ASR: del audio al texto

El reconocimiento automático del habla (ASR, *Automatic Speech Recognition*) responde **"¿qué se dijo?"**.

## 2.1 Las cuatro etapas del ASR

```
voz ──► forma de onda ──► espectrograma log-Mel ──► modelo neuronal ──► decodificación ──► "texto"
```

### 1. Forma de onda
El micrófono registra variaciones de presión del aire. Digitalizadas, son una secuencia de números: **16 000 valores por segundo** (16 kHz, un solo canal). Un segundo de audio es un vector de 16 000 números; es difícil sacar información de ahí directamente.

### 2. Espectrograma log-Mel
Se transforma la onda en una representación tiempo-frecuencia:

- Se corta el audio en **ventanas de 25 ms** que avanzan **cada 10 ms** (100 ventanas por segundo).
- En cada ventana se mide **cuánta energía hay en cada banda de frecuencia** (transformada de Fourier).
- Las frecuencias se agrupan en **bandas en escala Mel**, que imita la percepción humana: distinguimos mejor los cambios en frecuencias bajas que en altas. Whisper usa **80 bandas** (128 en su versión large-v3).
- Se toma el **logaritmo** de la energía, porque el oído percibe la intensidad de forma aproximadamente logarítmica.

El resultado es como una **imagen**: el eje horizontal es el tiempo, el vertical la frecuencia, y la intensidad de cada píxel es la energía.

### 3. Modelo neuronal
Un modelo **encoder-decoder Transformer**, como Whisper ([Radford et al., 2022](https://arxiv.org/abs/2212.04356)), lee esa imagen:
- el **encoder** convierte el espectrograma en una secuencia de representaciones;
- el **decoder** genera el texto **token a token**, como un modelo de lenguaje condicionado por el audio.

Whisper se entrenó con 680 000 horas de audio multilingüe etiquetado de forma débil (subtítulos de internet), lo que lo hace robusto a ruido, acentos e idiomas, incluido el español.

### 4. Decodificación
Se busca la secuencia de palabras más probable:
- **greedy**: en cada paso se elige el token más probable;
- **beam search**: se mantienen las *n* mejores hipótesis parciales y se elige la mejor secuencia completa. Es más lento pero suele dar mejor texto.

> **Idea clave:** el modelo nunca ve el audio "crudo": ve una imagen tiempo-frecuencia (el espectrograma) y la traduce a texto. Analogía: es como leer una partitura en lugar de escuchar la vibración del aire.

## 2.2 La rama ASR de DASR: VAD → transcribe → FA

Whisper transcribe bien, pero **sus marcas de tiempo son por segmento (frase) y poco precisas**. Para saber qué hablante dijo cada palabra necesitamos tiempos **por palabra**. Por eso la rama tiene tres pasos, que es el enfoque de **WhisperX** ([Bain et al., 2023](https://arxiv.org/abs/2303.00747)):

### Paso 1: VAD (detección de actividad de voz)
Detecta dónde hay voz para no transcribir silencio ni ruido (Whisper tiende a "inventar" texto en silencios largos, las llamadas *alucinaciones*).

En DASR **no se usa otro modelo**: se reutiliza la segmentación de la Fase 1. La regla es simple: **hay voz en un frame si algún hablante local está activo** (algún `b(w)f,k = 1`).

### Paso 2: transcribe
Whisper transcribe cada región de voz detectada. El resultado es texto con tiempos aproximados por segmento.

*Sobre la variante mencionada en el PPT ("generar los espectrogramas y pasarlos por un modelo de transcripción en lugar de usar directamente el audio"): Whisper ya calcula internamente el espectrograma log-Mel a partir del audio, así que con Whisper esa variante no cambia nada. Solo tendría sentido con un modelo de ASR distinto que reciba los rasgos ya calculados.*

### Paso 3: FA (alineación forzada)
Un **modelo fonético** — wav2vec 2.0 ([Baevski et al., 2020](https://arxiv.org/abs/2006.11477)) afinado con **CTC** (*Connectionist Temporal Classification*) — recibe **el audio y el texto ya conocido** y encuentra en qué instante exacto aparece cada palabra.

¿Por qué funciona? Un modelo CTC predice, en cada frame, la probabilidad de cada carácter (o de "nada"). Como el texto ya se conoce, solo hay que encontrar el camino más probable que recorra esos caracteres en orden a lo largo de los frames (programación dinámica, tipo Viterbi). Los frames donde empieza y termina cada palabra dan sus tiempos, con una precisión de decenas de milisegundos.

### Salida de la rama ASR

Una lista de palabras con inicio, fin y confianza:

```json
{"word": "lanzar", "start": 1.24, "end": 1.61, "score": 0.93}
{"word": "la",     "start": 1.61, "end": 1.70, "score": 0.97}
```

Esta lista es la que luego se cruza con la diarización ([archivo 7](07-integracion-asr-diarizacion.md)).

> **Idea clave:** sin tiempos precisos por palabra no podemos saber qué hablante dijo cada palabra.

## 2.3 WhisperX como referencia

WhisperX combina exactamente estas piezas: VAD + Whisper (vía faster-whisper, más rápido) + alineación forzada con wav2vec 2.0 + diarización con pyannote. Es la referencia de la rama ASR de DASR. La diferencia es que en DASR el VAD y la diarización salen de **nuestra** reproducción del pipeline, y no de llamar a pyannote como librería.
