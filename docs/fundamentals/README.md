# Estado del arte y marco teórico: proyecto DASR

**Reconocimiento automático del habla y diarización de hablantes: del audio a la transcripción por hablante y al análisis de atributos vocales.**

Este documento desarrolla por escrito la presentación *DASR – marco teórico* de Melissa Maureen. Sigue su mismo orden (cinco bloques, tres fases de diarización), amplía la explicación de cada diapositiva a partir de sus notas y añade lo que el formato de presentación no permite: más contexto, ejemplos resueltos, la correspondencia con el código de pyannote y el panorama de alternativas.

## La idea en una frase

Partimos del audio y obtenemos dos salidas que comparten el eje del tiempo: **qué se dijo** (ASR) y **quién habló y cuándo** (diarización). Las unimos palabra por palabra y, con la voz de cada hablante, estimamos **atributos vocales potenciales** (sexo percibido, grupo de edad, acento, estilo de habla).

La herramienta de referencia es [pyannote.audio](https://github.com/pyannote/pyannote-audio), pero **no la usamos como caja negra**: reproducimos su pipeline paso a paso en nuestro propio código (DASR) para poder explicar y verificar cada tensor intermedio.

## Por qué esta propuesta

La propuesta se eligió tras revisar papers de 2023 a 2026 sobre diarización, ASR y *fine-tuning* en este contexto. Los criterios fueron:

- **Trabajable** por el equipo en el tiempo del curso.
- **Cercana al curso**: redes neuronales, embeddings, clustering, métricas.
- **Comprensible**: algoritmos y decisiones que se pueden explicar sin cajas negras.
- **Moderna**: es la arquitectura de los sistemas abiertos más usados hoy (pyannote 3.x/4.0, WhisperX).
- **Con código público**: pyannote.audio está en GitHub, así que podemos ver cómo resuelven cada paso y comparar.

## Índice

| Bloque del PPT | Archivo | Qué responde |
|----------------|---------|--------------|
| Visión general | [1. Visión general y arquitectura](01-vision-general.md) | ¿Qué queremos obtener y cómo está organizado el código? |
| Fundamentos | [2. ASR: del audio al texto](02-asr.md) | ¿Qué se dijo? |
| Fundamentos | [3. Diarización: fundamentos](03-diarizacion-fundamentos.md) | ¿Quién habló y cuándo? ¿Por qué el enfoque de pyannote? |
| Diarización en 3 fases | [4. Fase 1: preparación](04-fase1-preparacion.md) | Segmentación local y segmentación binaria |
| Diarización en 3 fases | [5. Fase 2: procesamiento local](05-fase2-procesamiento-local.md) | Un embedding por hablante local, con su voz limpia |
| Diarización en 3 fases | [6. Fase 3: procesamiento global](06-fase3-procesamiento-global.md) | Clustering AHC con centroides y reconstrucción |
| Integración y atributos | [7. Fusión ASR + diarización](07-integracion-asr-diarizacion.md) | ¿Quién dijo cada palabra? |
| Integración y atributos | [8. Atributos vocales (AVA)](08-atributos-vocales.md) | ¿Cómo es la voz de cada hablante? |
| Implementación | [9. Evaluación y métricas](09-evaluacion.md) | ¿Cómo medimos si funciona? |
| Implementación | [10. Implementación y plan](10-implementacion-y-plan.md) | Del repositorio de pyannote a nuestros módulos |
| Complemento | [11. Fine-tuning](11-fine-tuning.md) | ¿Cómo adaptar los modelos a nuestros datos? |
| Complemento | [12. Contexto, alternativas y limitaciones](12-contexto-alternativas-limitaciones.md) | NeMo, WhisperX, DiariZen, Speech LLMs y límites actuales |
| — | [Referencias](referencias.md) | Bibliografía con enlaces |

Los archivos 11 y 12 no están en el PPT: recogen el resto de la revisión bibliográfica (fine-tuning, NeMo, limitaciones) para que el estado del arte quede completo.

## Notación usada en todo el documento

| Símbolo | Significado |
|---------|-------------|
| `w` | Índice de **ventana** (trozo de 10 s de audio). Hay `C` ventanas en total. |
| `f` | Índice de **frame** (≈17 ms). Cada ventana tiene `F` = 589 frames. |
| `k` | Índice de **hablante local** dentro de una ventana (`#1`, `#2`, `#3`). `K` = 3. |
| `c` | Índice de **hablante global** o clúster (`SPK_A`, `SPK_B`…). |
| `s(w)f,k` | Probabilidad (0–1) de que el hablante local `k` hable en el frame `f` de la ventana `w`. |
| `θ` | Umbral de binarización (solo para modelos multilabel). |
| `b(w)f,k` | Decisión binaria (0/1): ¿habla `k` en el frame `f` de `w`? |
| `m(w,k)` | Máscara de voz limpia del hablante local `k` en la ventana `w`. |
| `e(w,k)` | Embedding (huella vocal, 256 dimensiones) del hablante local `k` en la ventana `w`. |
| `πw(k)` | Asignación: a qué hablante global `c` corresponde el local `k` de la ventana `w`. |
| `δ` | Umbral de distancia que detiene el clustering aglomerativo. |
| `Kf` | Número de hablantes simultáneos estimado en el frame `f`. |
| `S(f,c)` | Puntuación del hablante global `c` en el frame `f`. |

## Figuras

Las figuras de la carpeta [img/](img/) provienen de la presentación. Las dos imágenes promocionales con marca comercial (el esquema de ASR de Deepgram y el banner de WhisperX) no se incluyen; su contenido se explica en texto.
