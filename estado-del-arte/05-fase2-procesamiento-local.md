# 5. Fase 2: procesamiento local

**Objetivo:** para cada ventana, extraer **un embedding por cada hablante local activo**. Un embedding es un vector (aquí de 256 números) que resume "cómo suena" una voz: voces de la misma persona dan vectores cercanos y voces distintas, vectores lejanos.

## 5.1 Enfoque clásico vs. propuesto

| | Enfoque clásico | Enfoque propuesto (pyannote) |
|---|----------------|------------------------------|
| **Qué se corta** | Segmentos cortos y **continuos** de longitud fija (≈1–2 s) | Toda la ventana de 10 s |
| **Cuántos embeddings** | Uno por segmento, hable quien hable | **Uno por cada hablante local activo** en cada ventana |
| **Qué audio usa** | Todo el segmento | Solo los frames donde `k` habla **y ningún otro** habla |
| **Problema** | Si dentro del segmento hay cambio de hablante o solapamiento, la huella se **contamina** (mezcla dos voces) | — |
| **Ventaja** | — | Aprovecha hasta 10 s de voz y **excluye el solapamiento** |

> **Idea clave:** más voz limpia por embedding = huella vocal más fiable = clustering más fácil.

## 5.2 La máscara de voz limpia

Para el hablante local `k` en la ventana `w` construimos una máscara que vale 1 solo en los frames donde `k` está activo **y todos los demás `k′` están inactivos**:

```
m(w,k)f = b(w)f,k · ∏(k′≠k) (1 − b(w)f,k′)
```

Leída por partes:
- `b(w)f,k` → vale 1 si `k` habla en el frame `f`;
- `(1 − b(w)f,k′)` → vale 1 si el otro hablante `k′` **no** habla;
- el producto `∏` exige que **ningún** otro hable.

Después se calcula el embedding **solo con el audio enmascarado**:

```
e(w,k) = Emb( x(w) ⊙ m(w,k) )
```

donde `x(w)` es el audio de la ventana y `⊙` es la multiplicación elemento a elemento. **Los frames no tienen que ser contiguos:** se concatena toda la voz limpia de `k` dentro de la ventana.

En el código de pyannote (`get_embeddings` con `exclude_overlap=True`) la condición se escribe como "la suma de `b` en el frame es menor que 2", que es equivalente: si `k` está activo y la suma es < 2, nadie más habla.

## 5.3 Ejemplo: cómo se construye la máscara

Ventana de 10 s con tres hablantes locales:

```
tiempo (s)   0    1    2    3    4    5    6    7    8    9   10
             |----|----|----|----|----|----|----|----|----|----|
Local #1       ██████████████████              ██████████
               0.5 ─────────── 3.5             6.0 ──── 8.0
Local #2                     ██████████████        ██████████
                             3.0 ──────── 5.5      7.5 ──── 9.5
Local #3       (inactivo → sin embedding)
Solapamiento                 ▓▓                    ▓▓
                           3.0–3.5               7.5–8.0   (excluido)

Máscara #1     ████████████████                ████████
               0.5 ──────── 3.0                6.0 ── 7.5     → 2.5 s + 1.5 s = 4.0 s de voz limpia
Máscara #2                      ████████████          ███████
                                3.5 ───── 5.5         8.0 ─ 9.5 → 2.0 s + 1.5 s = 3.5 s de voz limpia
```

- `e(w,1)` se calcula con **4 s** de voz limpia de `#1` (dos tramos concatenados).
- `e(w,2)` se calcula con **3.5 s** de voz limpia de `#2`.
- `#3` no habla en la ventana: **no genera embedding**.
- Las zonas de solapamiento (3.0–3.5 s y 7.5–8.0 s) **no se usan** para ningún embedding, porque mezclarían dos voces.

Con el enfoque clásico, en cambio, el segmento de 3 a 4 s contendría dos voces y su embedding sería una mezcla de `#1` y `#2`.

## 5.4 Detalles de implementación

| Detalle | Decisión |
|---------|----------|
| **Modelo de embedding** (receta 3.1) | **WeSpeaker ResNet34** ([Wang et al., 2022](https://arxiv.org/abs/2210.17016)), entrenado en VoxCeleb; produce vectores de **256 dimensiones**. En Hugging Face: `pyannote/wespeaker-voxceleb-resnet34-LM`. |
| **Respaldo si hay poca voz limpia** | Si después de quitar el solapamiento queda muy poca voz (menos de las muestras mínimas que necesita el modelo de embedding), **se usa la máscara completa** (con solapamiento) para no perder al hablante. |
| **Hablantes inactivos** | Su embedding queda como **NaN** y se ignora en el clustering. |
| **Resultado** | Un tensor **C × K × D** (ventanas × hablantes locales × 256). |

### ¿Por qué ResNet34 de WeSpeaker?

Es una CNN 2D sobre bancos de filtros (fbanks), entrenada para clasificar miles de hablantes de VoxCeleb con una pérdida de margen angular. Después del entrenamiento se descarta la capa de clasificación y se usa la penúltima capa como embedding. Es el estándar de pyannote 3.x: buena precisión y velocidad razonable. Otras opciones son ECAPA-TDNN, CAM++ o modelos basados en WavLM (ver [archivo 12](12-contexto-alternativas-limitaciones.md)).
