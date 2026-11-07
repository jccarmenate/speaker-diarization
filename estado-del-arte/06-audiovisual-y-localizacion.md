# 6. Audiovisual y localización: el "dónde"

Cuando la entrada es **video**, la imagen permite responder dónde está quien habla (qué cara) y, en principio, ayudar a la diarización.

## 6.1 Detección de hablante activo (ASD)

Para cada cara que aparece en el video, decidir **en cada frame si está hablando**. Combina la boca (labios) con el audio.

| Modelo | Año | Idea |
|--------|-----|------|
| **AVA-ActiveSpeaker** ([Roth et al.](https://arxiv.org/abs/1901.01342)) | 2019 | Dataset de referencia: películas con caras anotadas como hablando / no hablando. |
| **TalkNet** ([Tao et al.](https://arxiv.org/abs/2107.06592)) | 2021 | Codificadores visual (labios) y de audio + atención cruzada + contexto temporal largo. Muy usado y con código disponible. |
| **LoCoNet** ([Wang et al.](https://arxiv.org/abs/2301.08237)) | 2023 | Contexto largo (temporal) y corto (entre caras de la misma escena). Entre los mejores en AVA-ActiveSpeaker. |
| **Light-ASD** (Liao et al., CVPR 2023) | 2023 | Versión ligera, útil si no hay GPU potente. |

Pipeline típico para el "dónde" en video:

```
video → detección de caras → tracking de caras → ASD por cara → "la cara #2 habla de 00:13 a 00:21"
                                                            ↘ cruzar con la diarización de audio → SPK_1 = cara #2
```

Así cada etiqueta de hablante se asocia a una **cara y su posición** (el cuadro delimitador en cada frame). Si además se reconoce la cara, también se obtiene la identidad por la vía visual.

## 6.2 Diarización audiovisual

| Trabajo | Aporte |
|---------|--------|
| **AVA-AVD** ([Xu et al., 2021](https://arxiv.org/abs/2111.14448)) | Dataset y método de diarización audiovisual "en la naturaleza" (películas). Muestra que cuando el hablante está fuera de cámara, el audio sigue siendo indispensable. |
| **Ego4D AV diarization** | Diarización desde video en primera persona (gafas). pyannote 2.1 fue 1.º en 2022. |
| **MISP 2025** ([arXiv 2505.13971](https://arxiv.org/abs/2505.13971)) | Reto de Interspeech 2025 con 125 h de reuniones reales en audio + video. Baseline 15.52% de DER; ganador (DKU-WHU) 8.09% con un modelo seq2seq multicanal; 2.º XMUSPEECH 8.18% con atención cruzada audio-video ([CASA-Net](https://arxiv.org/abs/2506.02621)). |

**Hallazgo importante de MISP 2025:** los equipos punteros reportan que **la información visual no mejora significativamente** a un sistema solo-audio fuerte en reuniones. El video sirve sobre todo para:
- el **"dónde"** (qué persona en pantalla habla);
- casos difíciles: audio muy ruidoso o voces muy parecidas;
- asociar la voz con una **cara** para identificar visualmente.

## 6.3 Localización espacial (arreglos de micrófonos)

Con varios micrófonos se puede estimar la **dirección de llegada** (DOA) de cada voz (GCC-PHAT, SRP-PHAT, redes neuronales). Los sistemas de reuniones multicanal (CHiME-6/7/8, AliMeeting, MISP) usan esa información espacial para separar hablantes. **No aplica** a audios mono o videos comunes, que es el caso del proyecto; se menciona para completar.

## 6.4 Qué significa para el proyecto

- Si la entrada es **audio**: el "dónde" = marcas de tiempo (sale de la diarización).
- Si la entrada es **video**: añadir **ASD** (TalkNet o Light-ASD preentrenados) y unir sus resultados con la diarización de audio por solapamiento temporal. Es un módulo opcional y separable, ideal como extensión del proyecto.
