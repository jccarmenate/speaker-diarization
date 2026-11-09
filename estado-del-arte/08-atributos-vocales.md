# 8. Atributos vocales potenciales (AVA)

**Sección experimental.** Una vez que sabemos qué segmentos son de cada hablante global, analizamos **su voz** para estimar rasgos de la persona.

> **Idea clave:** son atributos **potenciales**: inferencias probabilísticas sobre la voz, **no la identidad declarada** de la persona.

## 8.1 Qué atributos y con qué rasgos

| Atributo | Rasgos que lo delatan | Cómo se estima |
|----------|----------------------|----------------|
| **Sexo de la voz (percibido)** | **F0 media** (frecuencia fundamental, el "tono"): rangos típicos de adultos ~85–180 Hz y ~165–255 Hz. **Formantes F1–F3**, que reflejan la longitud del tracto vocal. | Clasificador simple sobre los embeddings, que ya codifican esta información |
| **Grupo de edad** | **Jitter** (inestabilidad de la frecuencia de un ciclo a otro), **shimmer** (inestabilidad de la amplitud), **HNR** (relación armónico-ruido), F0 y velocidad de habla | Clasificación por rangos (joven / adulto / mayor) o regresión en años |
| **Acento / variedad dialectal** | Rasgos fonéticos y prosódicos (MFCC, ritmo, entonación) | Modelos autosupervisados (wav2vec 2.0, ECAPA) **afinados** con datos etiquetados como Common Voice |
| **Estilo y fluidez** | Palabras por minuto y pausas (desde el ASR); tiempo de habla, turnos e interrupciones (desde la diarización) | Se **calcula directamente** de nuestras salidas, sin modelo adicional |

### Glosario rápido
- **F0:** frecuencia de vibración de las cuerdas vocales. Es lo que percibimos como voz "grave" o "aguda".
- **Formantes (F1, F2, F3):** frecuencias que el tracto vocal (garganta, boca) refuerza. Dependen de su tamaño y forma.
- **Jitter / shimmer:** pequeñas irregularidades ciclo a ciclo en frecuencia y amplitud. Suelen aumentar con la edad.
- **HNR:** cuánta energía es armónica (voz "limpia") frente a ruido (voz "rasposa" o aireada).
- **MFCC:** coeficientes que resumen la forma del espectro; un rasgo clásico de reconocimiento de voz.

## 8.2 Pipeline del módulo AVA

```
ENTRADA (hablante c)          REPRESENTACIÓN                      MODELOS                       SALIDA
─────────────────────         ──────────────────────────          ─────────────────────         ──────────────────────
Turnos y audio de c     ──►   Centroide μc (voz promedio)   ──►   Sondas lineales sobre   ──►   Atributo + probabilidad
Embeddings e(w,k)             Rasgos acústicos: F0, formantes,    embeddings (regresión          por hablante
  con πw(k) = c               jitter, shimmer, HNR, MFCC          logística, SVM)               Visualización t-SNE/UMAP
Palabras y tiempos            Rasgos de habla: palabras/min,      MLP sobre rasgos combinados    por atributo
  del ASR                     pausas                              Modelos SSL afinados          Informe por conversación
                                                                  (wav2vec 2.0)
```

1. **Entrada:** para cada hablante global `c`, sus turnos, sus embeddings locales (los `e(w,k)` con `πw(k) = c`) y sus palabras.
2. **Representación:** el **centroide** `μc` resume su voz; se añaden rasgos acústicos interpretables y rasgos de habla calculados desde el ASR.
3. **Modelos**, de más simple a más complejo:
   - una **sonda** (*probe*): un clasificador lineal sobre los embeddings. Es lo más simple y, según la literatura, sorprendentemente efectivo;
   - una red pequeña (**MLP**) que combina embeddings y rasgos;
   - **afinar modelos autosupervisados** (wav2vec 2.0) para el atributo.
4. **Salida:** el atributo **con su probabilidad**, visualizaciones del espacio de embeddings coloreadas por atributo y un informe por conversación.

> **Idea clave:** el embedding se entrenó para distinguir **personas**, pero también guarda sexo, edad y acento: basta con "sondearlo".

## 8.3 Qué dice la literatura

La revisión bibliográfica encontró que **el análisis de atributos es el área con menos resultados significativos**, con una excepción: **entrenar (o afinar) modelos específicos para reconocer cada atributo**. Los trabajos que sustentan la propuesta:

| Trabajo | Qué muestra | Relevancia para AVA |
|---------|-------------|---------------------|
| [Raj et al., 2019 — *Probing the Information Encoded in X-vectors*](https://arxiv.org/abs/1909.06351) | Los embeddings de hablante (x-vectors) codifican, además de la identidad, sexo, canal, contenido léxico y más; se recuperan con clasificadores simples sobre ellos. | Justifica el uso de **sondas lineales** sobre nuestros embeddings. |
| [Burkhardt et al., 2023 — *Speech-based Age and Gender Prediction with Transformers*](https://arxiv.org/abs/2306.16962) (audEERING) | wav2vec 2.0 afinado para predecir edad (regresión) y sexo; publican el modelo. | Referencia de **edad y sexo** con modelos SSL afinados. |
| [Zuluaga-Gomez et al., 2023 — *CommonAccent*](https://arxiv.org/abs/2305.18283) | Clasificación de acentos afinando modelos preentrenados (XLSR, ECAPA) sobre Common Voice. | Referencia para **acento**; usa el mismo corpus que proponemos. |
| [Ardila et al., 2020 — *Common Voice*](https://arxiv.org/abs/1912.06670) | Corpus multilingüe con metadatos de edad, sexo y acento declarados por los hablantes. | **Datos etiquetados** para entrenar y evaluar AVA, incluido español. |

**Lectura honesta del estado del arte:**
- **Sexo percibido:** funciona bien con casi cualquier enfoque (incluso una sonda lineal).
- **Edad:** más difícil; los errores típicos son de varios años y dependen mucho del dominio.
- **Acento:** depende fuertemente de tener datos etiquetados de las variedades que interesan.
- **Estilo y fluidez:** es lo más fiable, porque se mide directamente de las salidas del pipeline, sin inferir nada sobre la persona.

## 8.4 Validación y ética

### Validación correcta
- **Particiones por hablante:** el mismo hablante **nunca** puede estar en entrenamiento y en prueba. Si lo está, el modelo aprende a reconocer a la persona y no el atributo, y el rendimiento se **sobreestima**.
- Usar datos con metadatos como **Common Voice** (edad, sexo, acento).
- Métricas: **UAR** (recall promedio por clase, robusto al desbalance), **F1** y **MAE** en años para la edad ([archivo 9](09-evaluacion.md)).

### Responsabilidad
- Reportar siempre **probabilidades**, no etiquetas categóricas presentadas como hechos.
- **Evaluar por subgrupos** (sexo, edad, acento) para detectar sesgos: un modelo puede funcionar bien en promedio y mal en un grupo minoritario.
- El "sexo de la voz" es una **percepción acústica**, no el género de la persona.
