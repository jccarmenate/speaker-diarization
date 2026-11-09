# 9. Evaluación: ¿cómo medimos si funciona?

Cada rama del pipeline tiene su métrica. Como las ramas son independientes, cada una se evalúa por separado.

## 9.1 ASR: WER

```
WER = (S + D + I) / N
```

| Término | Significado |
|---------|-------------|
| `S` | **Sustituciones**: palabra reconocida distinta de la de referencia |
| `D` | **Borrados** (*deletions*): palabras de la referencia que el sistema omitió |
| `I` | **Inserciones**: palabras que el sistema añadió y no estaban |
| `N` | Número de palabras de la referencia |

**Ejemplo.** Referencia: "deberíamos lanzar la beta" (N = 4). Hipótesis: "deberíamos lanzar beta ya".
Un borrado ("la") y una inserción ("ya") → WER = (0 + 1 + 1) / 4 = **50 %**.

Menor es mejor. Puede superar el 100 % si hay muchas inserciones. Se calcula con librerías como `jiwer`. Antes de comparar hay que **normalizar el texto** (minúsculas, sin puntuación), o el WER penaliza diferencias que no importan.

## 9.2 Diarización: DER

```
DER = (FA + Miss + Conf) / T
```

| Término | Significado |
|---------|-------------|
| `FA` | **Falsa alarma**: tiempo en que el sistema dice que alguien habla y nadie habla |
| `Miss` | **Voz perdida**: tiempo en que alguien habla y el sistema no lo detecta |
| `Conf` | **Confusión**: tiempo asignado al hablante equivocado |
| `T` | Tiempo total de habla en la referencia |

Para calcular `Conf`, primero se busca la mejor correspondencia entre las etiquetas del sistema (`SPK_A`…) y las de la referencia (algoritmo húngaro), porque las etiquetas son anónimas.

**Cómo se calcula importa.** Al comparar cifras hay que mirar:
- **collar:** margen sin evaluar alrededor de cada frontera (0 s o 0.25 s). Con collar el DER baja;
- si se **cuenta el solapamiento** o se ignora.

Las cifras de pyannote usan la configuración más estricta: **sin collar, contando el solapamiento** y con detección de voz automática.

También se puede reportar el **JER** (*Jaccard Error Rate*), que pondera a todos los hablantes por igual (en el DER, los que hablan poco casi no cuentan).

Herramienta: `pyannote.metrics`.

## 9.3 Atributos: UAR, F1, MAE

| Métrica | Para qué | Por qué |
|---------|----------|---------|
| **UAR** (*Unweighted Average Recall*) | Clasificación (sexo, grupo de edad, acento) | Promedia el recall de cada clase por igual. Si el 90 % de los datos es de una clase, la exactitud engaña; el UAR no. |
| **F1** | Clasificación | Equilibrio entre precisión y recall. |
| **MAE** (*Mean Absolute Error*) | Edad como regresión | Error medio en años: "nos equivocamos en ±X años". |

## 9.4 Valores de referencia: DER publicado por pyannote

DER (%), menor es mejor. Fuente: ficha del modelo `pyannote/speaker-diarization-community-1` (benchmark de septiembre 2025).

| Dataset | Tipo de audio | pyannote 3.1 (AHC, nuestra receta) | community-1 (4.0, VBx) |
|---------|---------------|-----------------------------------:|-----------------------:|
| AISHELL-4 | Reuniones en mandarín | 12.2 | 11.7 |
| AliMeeting | Reuniones en mandarín | 24.5 | 20.3 |
| AMI IHM | Reuniones en inglés, micrófono de cabeza | 18.8 | 17.0 |
| AMI SDM | Reuniones en inglés, un micrófono lejano | 22.7 | 19.9 |
| AVA-AVD | Películas | 49.7 | 44.6 |
| CALLHOME | Llamadas telefónicas, multilingüe | 28.5 | 26.7 |
| DIHARD 3 | 11 dominios difíciles | 21.4 | 20.2 |

**Cómo leer la tabla:**
- La columna **3.1** es nuestra vara de medir: reproducir el pipeline debe acercarnos a esos DER en los mismos datos.
- La diferencia entre columnas muestra cuánto aporta cambiar AHC por VBx (y otras mejoras de la 4.0): entre 0.5 y 5 puntos según el dataset.
- El DER **depende mucho del tipo de audio**: AMI con micrófono de cabeza es mucho más fácil que AMI con un micrófono lejano; las películas (AVA-AVD) son el caso más difícil.

> **Idea clave:** la tabla de la receta 3.1 es nuestra vara de medir: reproducir el pipeline debe acercarnos a esos DER.
