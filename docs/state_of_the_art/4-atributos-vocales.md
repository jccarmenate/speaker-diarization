# Frente 4: atributos vocales a partir de la voz

Revisión bibliográfica para DASR: qué atributos de la voz (sexo percibido, edad, acento, estilo) se pueden estimar, con qué entradas y con qué fiabilidad. Fecha de la búsqueda: 12 de noviembre de 2026. Autores, año, título y enlace de cada referencia se comprobaron contra arXiv o Crossref. Las venues que en arXiv solo constan como "enviado" o que no se pudieron confirmar van marcadas "(no verificado)". El resumen de cada trabajo se basa en su abstract, salvo cuando se indica otra cosa.

Leyenda de etiquetas: [atributos] qué estima, [entrada] rasgos acústicos, embeddings o modelos autosupervisados, [datos], [código], [relevancia] para DASR.

Los PDF de este banco están en [`pdfs/4-atributos-vocales/`](pdfs/4-atributos-vocales/); el [índice](pdfs/INDICE.md) lista los que hay que bajar a mano.

## 1. Qué información guardan las representaciones

### 1.1 Sondeo de x-vectors
- Referencia: Desh Raj, David Snyder, Daniel Povey, Sanjeev Khudanpur (2019). "Probing the Information Encoded in X-vectors". ASRU 2019. arXiv: https://arxiv.org/abs/1909.06351. DOI: https://doi.org/10.1109/ASRU46091.2019.9003979 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Sondea x-vectors con clasificadores simples para ver qué codifican: información del hablante, del canal, de la transcripción (frase, palabras, fonemas) y metadatos de la grabación (duración, tipo de aumento), y los compara con i-vectors. Los x-vectors capturan contenido hablado e información de canal además de servir para verificar hablantes.
- Relevancia para DASR: Justifica el uso de sondas lineales sobre los embeddings de DASR, y advierte de que también codifican canal y contenido, que pueden confundir a un clasificador de atributos.
- Etiquetas: [atributos: hablante, canal, contenido, metadatos] [entrada: embeddings] [datos: RedDots] [código: no localizado] [relevancia: alta]
- PDF: [1.01-Raj-2019-Probing-the-Information-Encoded-in-X-vectors.pdf](pdfs/4-atributos-vocales/1.01-Raj-2019-Probing-the-Information-Encoded-in-X-vectors.pdf)

### 1.2 Sondeo a gran escala de modelos autosupervisados
- Referencia: Aemon Yat Fei Chiu, Kei Ching Fung, Roger Tsz Yeung Li, Jingyu Li, Tan Lee (2025). "A Large-Scale Probing Analysis of Speaker-Specific Attributes in Self-Supervised Speech Representations". ISCSLP 2026. arXiv: https://arxiv.org/abs/2501.05310.
- Qué hace: Sondeo capa por capa de 11 modelos autosupervisados para identidad del hablante y atributos acústicos, prosódicos y paralingüísticos. Las primeras capas codifican la acústica básica y las intermedias los rasgos abstractos; para varios atributos, las capas intermedias dan mejores sondas que los embeddings de hablante especializados.
- Relevancia para DASR: Sugiere comparar en AVA el centroide ResNet34 con las capas intermedias de un modelo autosupervisado como WavLM.
- Etiquetas: [atributos: acústicos, prosódicos, paralingüísticos, identidad] [entrada: SSL por capa] [datos: varios] [código: no localizado] [relevancia: alta]
- PDF: [1.02-Chiu-2025-A-Large-Scale-Probing-Analysis-of-Speaker-Specific.pdf](pdfs/4-atributos-vocales/1.02-Chiu-2025-A-Large-Scale-Probing-Analysis-of-Speaker-Specific.pdf)

## 2. Edad y sexo

### 2.1 Reto paralingüístico de Interspeech 2010
- Referencia: Björn Schuller, Stefan Steidl, Anton Batliner, Felix Burkhardt, Laurence Devillers, Christian Müller, Shrikanth S. Narayanan (2010). "The INTERSPEECH 2010 paralinguistic challenge". Interspeech 2010. DOI: https://doi.org/10.21437/interspeech.2010-739.
- Qué hace: Reto con subtareas de edad y sexo del hablante, evaluadas con recall promedio no ponderado (UAR), con un conjunto de rasgos acústicos de referencia.
- Relevancia para DASR: Referencia histórica de la tarea de edad y sexo y del uso de UAR como métrica.
- Etiquetas: [atributos: edad, sexo] [entrada: rasgos acústicos] [datos: aGender] [código: n/a] [relevancia: media]
- PDF: [2.01-Schuller-2010-The-INTERSPEECH-2010-paralinguistic-challenge.pdf](pdfs/4-atributos-vocales/2.01-Schuller-2010-The-INTERSPEECH-2010-paralinguistic-challenge.pdf)

### 2.2 Edad y sexo con redes profundas
- Referencia: Damian Kwasny, Daria Hemmerling (2021). "Gender and Age Estimation Methods Based on Speech Using Deep Neural Networks". Sensors 21(14). DOI: https://doi.org/10.3390/s21144785.
- Qué hace: Estudia métodos de estimación de edad y sexo a partir de la voz con redes neuronales profundas (no verificado el detalle de arquitecturas y resultados).
- Relevancia para DASR: Referencia de acceso abierto sobre el enfoque clásico de edad y sexo con redes profundas.
- Etiquetas: [atributos: edad, sexo] [entrada: no verificado] [datos: no verificado] [código: no localizado] [relevancia: baja]
- PDF: descarga manual (acceso abierto, pero MDPI bloquea las descargas automáticas: bajarlo desde el navegador); ver [`pdfs/INDICE.md`](pdfs/INDICE.md)

### 2.3 VoxCeleb con edad y sexo
- Referencia: Khaled Hechmi, Trung Ngo Trong, Ville Hautamaki, Tomi Kinnunen (2021). "VoxCeleb Enrichment for Age and Gender Recognition". ASRU 2021. arXiv: https://arxiv.org/abs/2109.13510.
- Qué hace: Añade etiquetas de edad y una anotación alternativa de sexo a VoxCeleb consultando bases de datos de celebridades. Su mejor sistema logra F1 de 0.9829 en sexo y un error absoluto medio de 9.443 años en edad.
- Relevancia para DASR: Da etiquetas para entrenar sondas de AVA sobre el mismo dominio con el que se entrenó el embedding de la Fase 2, y fija una referencia de MAE.
- Etiquetas: [atributos: edad, sexo] [entrada: i-vectors, x-vectors y otros] [datos: VoxCeleb] [código: https://github.com/hechmik/voxceleb_enrichment_age_gender] [relevancia: alta]
- PDF: [2.03-Hechmi-2021-VoxCeleb-Enrichment-for-Age-and-Gender-Recognition.pdf](pdfs/4-atributos-vocales/2.03-Hechmi-2021-VoxCeleb-Enrichment-for-Age-and-Gender-Recognition.pdf)

### 2.4 Edad y sexo con transformers
- Referencia: Felix Burkhardt, Johannes Wagner, Hagen Wierstorf, Florian Eyben, Björn Schuller (2023). "Speech-based Age and Gender Prediction with Transformers". ITG Speech Communication 2023 (no verificado). arXiv: https://arxiv.org/abs/2306.16962.
- Qué hace: Cura varios datasets públicos y predice edad y sexo con modelos basados en wav2vec 2.0: MAE de 7.1 a 10.8 años según el dataset y al menos 91.1 % de exactitud en sexo (mujer, hombre, niño). Mejora en UAR a un sistema con rasgos manuales y publica el modelo.
- Relevancia para DASR: Modelo listo para usar como referencia de AVA en edad y sexo.
- Etiquetas: [atributos: edad, sexo] [entrada: wav2vec 2.0] [datos: varios públicos] [código: modelo audeering/wav2vec2-large-robust-24-ft-age-gender en Hugging Face (no verificado)] [relevancia: alta]
- PDF: [2.04-Burkhardt-2023-Speech-based-Age-and-Gender-Prediction-with-Transformers.pdf](pdfs/4-atributos-vocales/2.04-Burkhardt-2023-Speech-based-Age-and-Gender-Prediction-with-Transformers.pdf)

### 2.5 Atributos demográficos con WavLM
- Referencia: Yuchen Yang, Thomas Thebaud, Najim Dehak (2025). "Demographic Attributes Prediction from Speech Using WavLM Embeddings". CISS 2025 (no verificado). arXiv: https://arxiv.org/abs/2502.12007.
- Qué hace: Clasificador general sobre rasgos de WavLM que predice edad, sexo, lengua materna, nivel educativo y país. Reporta MAE de 4.94 años en edad y más de 99.81 % de exactitud en sexo, con mejoras de hasta 30 % relativo en MAE.
- Relevancia para DASR: Es la referencia cuantitativa más fuerte para AVA y apoya usar WavLM como entrada.
- Etiquetas: [atributos: edad, sexo, lengua, educación, país] [entrada: WavLM] [datos: varios] [código: no localizado] [relevancia: alta]
- PDF: [2.05-Yang-2025-Demographic-Attributes-Prediction-from-Speech-Using-WavLM-Embeddings.pdf](pdfs/4-atributos-vocales/2.05-Yang-2025-Demographic-Attributes-Prediction-from-Speech-Using-WavLM-Embeddings.pdf)

## 3. Acento

### 3.1 CommonAccent
- Referencia: Juan Zuluaga-Gomez, Sara Ahmed, Danielius Visockas, Cem Subakan (2023). "CommonAccent: Exploring Large Acoustic Pretrained Models for Accent Classification Based on Common Voice". Interspeech 2023. arXiv: https://arxiv.org/abs/2305.18283.
- Qué hace: Clasificación multilingüe de acentos con ECAPA-TDNN y wav2vec 2.0/XLSR afinados sobre Common Voice (inglés, italiano, alemán y español), con una receta de SpeechBrain; hasta 95 % de exactitud en inglés.
- Relevancia para DASR: Referencia directa para el acento en AVA e incluye español.
- Etiquetas: [atributos: acento] [entrada: ECAPA, XLSR] [datos: Common Voice] [código: receta de SpeechBrain] [relevancia: alta]
- PDF: [3.01-ZuluagaGomez-2023-CommonAccent-Exploring-Large-Acoustic-Pretrained-Models-for-Accent.pdf](pdfs/4-atributos-vocales/3.01-ZuluagaGomez-2023-CommonAccent-Exploring-Large-Acoustic-Pretrained-Models-for-Accent.pdf)

### 3.2 Common Voice
- Referencia: Rosana Ardila, Megan Branson, Kelly Davis, Michael Henretty, Michael Kohler, Josh Meyer, Reuben Morais, Lindsay Saunders et al. (2019). "Common Voice: A Massively-Multilingual Speech Corpus". LREC 2020. arXiv: https://arxiv.org/abs/1912.06670.
- Qué hace: Corpus multilingüe de voz leída recogido por colaboración abierta, con metadatos opcionales de edad, sexo y acento declarados por los hablantes.
- Relevancia para DASR: Fuente de datos etiquetados para entrenar y evaluar AVA, incluido el español.
- Etiquetas: [atributos: edad, sexo, acento (metadatos)] [entrada: n/a] [datos: corpus] [código: https://commonvoice.mozilla.org] [relevancia: alta]
- PDF: [3.02-Ardila-2019-Common-Voice-A-Massively-Multilingual-Speech-Corpus.pdf](pdfs/4-atributos-vocales/3.02-Ardila-2019-Common-Voice-A-Massively-Multilingual-Speech-Corpus.pdf)

## 4. Rasgos acústicos interpretables

### 4.1 openSMILE
- Referencia: Florian Eyben, Martin Wöllmer, Björn Schuller (2010). "Opensmile". ACM Multimedia 2010. DOI: https://doi.org/10.1145/1873951.1874246.
- Qué hace: Extractor de rasgos de audio de código abierto (F0, energía, MFCC, jitter, shimmer y muchos más) muy usado en paralingüística.
- Relevancia para DASR: Herramienta para calcular los rasgos acústicos que el módulo AVA combina con los embeddings.
- Etiquetas: [atributos: n/a] [entrada: rasgos acústicos] [datos: n/a] [código: https://github.com/audeering/opensmile] [relevancia: media]
- PDF: descarga manual (de pago o sin copia libre localizada); ver [`pdfs/INDICE.md`](pdfs/INDICE.md)

### 4.2 eGeMAPS
- Referencia: Florian Eyben, Klaus R. Scherer, Bjorn W. Schuller, Johan Sundberg, Elisabeth Andre, Carlos Busso, Laurence Y. Devillers, Julien Epps et al. (2016). "The Geneva Minimalistic Acoustic Parameter Set (GeMAPS) for Voice Research and Affective Computing". IEEE Transactions on Affective Computing 7(2). DOI: https://doi.org/10.1109/taffc.2015.2457417.
- Qué hace: Conjunto mínimo y estandarizado de parámetros acústicos para investigación de la voz (F0, jitter, shimmer, HNR, formantes, entre otros).
- Relevancia para DASR: Lista de referencia de rasgos interpretables para AVA, disponible en openSMILE.
- Etiquetas: [atributos: n/a] [entrada: rasgos acústicos] [datos: n/a] [código: openSMILE] [relevancia: media]
- PDF: descarga manual (de pago o sin copia libre localizada); ver [`pdfs/INDICE.md`](pdfs/INDICE.md)

## 5. Estilo y paralingüística

### 5.1 Reto de emociones de Interspeech 2009
- Referencia: Björn Schuller, Stefan Steidl, Anton Batliner (2009). "The INTERSPEECH 2009 emotion challenge". Interspeech 2009. DOI: https://doi.org/10.21437/interspeech.2009-103.
- Qué hace: Primer reto de reconocimiento de emociones en el habla; consolida el UAR como métrica para clases desbalanceadas.
- Relevancia para DASR: Origen de la métrica UAR que DASR propone para los atributos.
- Etiquetas: [atributos: emoción] [entrada: rasgos acústicos] [datos: FAU Aibo] [código: n/a] [relevancia: baja]
- PDF: [5.01-Schuller-2009-The-INTERSPEECH-2009-emotion-challenge.pdf](pdfs/4-atributos-vocales/5.01-Schuller-2009-The-INTERSPEECH-2009-emotion-challenge.pdf)

### 5.2 Emociones con wav2vec 2.0
- Referencia: Leonardo Pepino, Pablo Riera, Luciana Ferrer (2021). "Emotion Recognition from Speech Using Wav2vec 2.0 Embeddings". Interspeech 2021 (no verificado). arXiv: https://arxiv.org/abs/2104.03502.
- Qué hace: Reconoce emociones con rasgos de wav2vec 2.0 preentrenado y redes simples, combinando varias capas con pesos aprendidos.
- Relevancia para DASR: Muestra la técnica de combinar capas de un modelo autosupervisado con pesos aprendidos, aplicable a las sondas de AVA.
- Etiquetas: [atributos: emoción] [entrada: wav2vec 2.0 por capas] [datos: IEMOCAP, RAVDESS] [código: https://github.com/habla-liaa/ser-with-w2v2] [relevancia: baja]
- PDF: [5.02-Pepino-2021-Emotion-Recognition-from-Speech-Using-Wav2vec-2-0.pdf](pdfs/4-atributos-vocales/5.02-Pepino-2021-Emotion-Recognition-from-Speech-Using-Wav2vec-2-0.pdf)

## 6. Sesgos

### 6.1 Sesgo en reconocimiento automático de hablantes
- Referencia: Wiebke Toussaint Hutiri, Aaron Ding (2022). "Bias in Automated Speaker Recognition". ACM FAccT 2022. arXiv: https://arxiv.org/abs/2201.09486. DOI: https://doi.org/10.1145/3531146.3533089 Focus to learn more DOI(s) linking to related resources.
- Qué hace: Estudio empírico del sesgo en verificación de hablantes a lo largo del desarrollo (datos, modelo, implementación) en el reto VoxCeleb; encuentra sesgo en todas las etapas, con más impacto en mujeres y en hablantes no estadounidenses.
- Relevancia para DASR: Justifica evaluar AVA por subgrupos y reportar probabilidades en lugar de etiquetas.
- Etiquetas: [atributos: n/a] [entrada: embeddings] [datos: VoxCeleb] [código: no localizado] [relevancia: media]
- PDF: [6.01-Hutiri-2022-Bias-in-Automated-Speaker-Recognition.pdf](pdfs/4-atributos-vocales/6.01-Hutiri-2022-Bias-in-Automated-Speaker-Recognition.pdf)

## Respuesta directa

La literatura confirma lo que encontró la revisión del equipo: el análisis de atributos tiene pocos resultados sólidos fuera de los modelos entrenados o afinados para cada atributo. El sexo percibido es fácil (más del 99 % con WavLM); la edad es difícil, con errores medios de entre 5 y 11 años según el modelo y el dataset; el acento depende de tener datos etiquetados de las variedades que interesan. Los embeddings de hablante sí contienen información de atributos, pero también de canal y contenido; las capas intermedias de modelos autosupervisados son mejores entradas que los embeddings especializados.

## Huecos detectados

1. No se encontraron resultados de edad, sexo o acento a partir de los embeddings de un pipeline de diarización (centroides por hablante global), que es la entrada que usa AVA.
2. Faltan evaluaciones en español de edad y acento con particiones por hablante.
3. No hay estudios del efecto de los errores de diarización (hablantes mezclados en un mismo clúster) sobre la estimación de atributos.
