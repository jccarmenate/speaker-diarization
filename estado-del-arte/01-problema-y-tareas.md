# 1. Problema y tareas

"Quién dijo qué, cuándo y dónde" no es una sola tarea de ML. Se descompone en varias, cada una con su literatura, sus datasets y sus métricas.

## Las tareas

| Pregunta | Tarea | Entrada → salida |
|----------|-------|------------------|
| ¿Hay voz? | **VAD** (*Voice Activity Detection*) | audio → segmentos con voz / sin voz |
| ¿Cuándo habla cada uno? | **Diarización** (*speaker diarization*) | audio → segmentos con etiqueta anónima (`SPK_0`, `SPK_1`…) |
| ¿Hablan a la vez? | **Detección de solapamiento** (*overlap detection*) | audio → segmentos con 2+ hablantes |
| ¿Quién es? | **Identificación de hablante** (*speaker identification*) | segmento + voces registradas → nombre o "desconocido" |
| ¿Es la misma persona? | **Verificación de hablante** (*speaker verification*) | dos segmentos → sí / no (puntuación) |
| ¿Qué dijo? | **ASR** (*Automatic Speech Recognition*) | audio → texto con marcas de tiempo |
| ¿Quién dijo cada palabra? | **ASR atribuido** (*speaker-attributed ASR*) | audio → texto con hablante por palabra |
| ¿Dónde está quien habla? | **Detección de hablante activo** (ASD) / **localización de fuente** | video o micrófonos → cara o dirección del hablante |

## Diarización vs. identificación

Se confunden a menudo, pero no son lo mismo:

- **Diarización** responde "quién habló cuándo" con etiquetas **anónimas**. No sabe nombres; solo agrupa los segmentos de la misma voz. No necesita conocer a los hablantes de antemano.
- **Identificación** pone **nombres**. Necesita un registro previo (*enrollment*): unos segundos de voz de cada persona conocida. Puede ser:
  - **conjunto cerrado** (*closed-set*): el hablante siempre es uno de los registrados;
  - **conjunto abierto** (*open-set*): puede aparecer alguien no registrado y el sistema debe decir "desconocido". Es el caso realista.

En la práctica se encadenan: primero se diariza (`SPK_0`, `SPK_1`) y después se compara el embedding de cada grupo contra las voces registradas para ponerle nombre.

## Las tres familias de enfoques

1. **Modular / en cascada** ([subtema 2](02-pipeline-modular.md)): VAD → embeddings por segmento → clustering. Es el enfoque clásico, interpretable y fácil de depurar.
2. **End-to-end** ([subtema 3](03-end-to-end.md)): una red neuronal predice directamente qué hablantes están activos en cada frame. Maneja bien el solapamiento.
3. **Híbrido**: segmentación *end-to-end* en ventanas cortas + clustering global. Es lo que usan los mejores sistemas abiertos hoy (pyannote 3.x, DiariZen).

Y, por encima de las tres, la tendencia más reciente: **modelos conjuntos de diarización + ASR** ([subtema 5](05-diarizacion-y-transcripcion.md)).

## El "dónde": tres interpretaciones

La palabra "dónde" del objetivo del proyecto admite tres lecturas. Conviene decidir cuál se aborda:

1. **Dónde en el archivo:** el minuto y segundo. Sale gratis de la diarización (marcas de tiempo).
2. **Dónde en la imagen:** en un video, qué cara o persona en pantalla está hablando. Es la **detección de hablante activo** ([subtema 6](06-audiovisual-y-localizacion.md)).
3. **Dónde en el espacio:** la dirección física del hablante respecto a los micrófonos (*sound source localization*, DOA). Necesita grabaciones con arreglos de varios micrófonos; poco aplicable a audios o videos comunes.

Para un proyecto que recibe "audio o video", lo más razonable es (1) siempre y (2) cuando la entrada es video.

## Referencia general

- Park et al., [*A Review of Speaker Diarization: Recent Advances with Deep Learning*](https://arxiv.org/abs/2101.09624) (2021). Sigue siendo la revisión más citada del área; no cubre lo posterior a 2021 (powerset, Sortformer, Speech LLMs), que se trata en los subtemas siguientes.
