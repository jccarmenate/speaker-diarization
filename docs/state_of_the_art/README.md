# State of the Art

This directory contains the systematic literature review that supports the design decisions of the DASR pipeline.

## Contents

| File | Description |
|---|---|
| `state-of-the-art.pdf` | Literature review covering ASR, speaker diarization, and vocal attribute analysis. |
| `state-of-the-art.tex` | LaTeX source of the literature review. The compiled PDF is `state-of-the-art.pdf`. |
| `sota.xlsx` | Comparative matrix of the reviewed papers. |

## SOTA

SOTA is a **comparative matrix** that evaluates each reviewed paper against the four Research Questions (RQ1–RQ4) that structure the project:

- **RQ1 · Segmentation/VAD:** Who speaks and when (overlap handling, temporal resolution).
- **RQ2 · Diarization & Embeddings:** How voices are grouped (embedding models, clustering).
- **RQ3 · ASR & Alignment:** What is said and who said it (ASR, forced alignment, word-to-speaker mapping).
- **RQ4 · Vocal Attributes:** How the voice is characterized (age, gender, accent, SSL probing).

### Structure

The workbook has three sheets, each with a distinct role:

| Sheet | Purpose |
|---|---|
| **Matriz** | Paper-by-paper comparison. Each row represents a paper, and each column represents a sub-question of RQ1–RQ4, a technique, or an evaluation metric. |
| **Codebook** | Defines every column and the criteria for assigning ✓ or ✗. It is the reference for interpreting the matrix. |
| **Evidencia** | Textual quotes and page numbers supporting each ✓, making the matrix auditable. |

### How to Read the Matrix

1. Start with the **Codebook** to understand what each column means and when to mark ✓ or ✗.
2. Read the **Matriz** to compare papers across RQ1–RQ4, techniques, and evaluation metrics.
3. Check the **Evidencia** to verify each ✓ using its supporting quote and page number.

## Reference banks

Besides the paper and its matrix, this folder holds a reference review organised in five banks (written in Spanish). Each entry gives the full reference, what the work does, why it matters for DASR and its relevance.

| File | Description |
|---|---|
| [`sintesis.md`](sintesis.md) | Synthesis: answers to the starting questions, baselines, notes on the pyannote 3.1.1 code, transferable ideas, gaps and a reading path. |
| [`1-diarizacion.md`](1-diarizacion.md) | Speaker diarization systems (27 entries). |
| [`2-representaciones-y-clustering.md`](2-representaciones-y-clustering.md) | Speaker embeddings, self-supervised models and clustering (22 entries). |
| [`3-asr-y-atribucion.md`](3-asr-y-atribucion.md) | ASR, forced alignment and joint ASR + diarization (17 entries). |
| [`4-atributos-vocales.md`](4-atributos-vocales.md) | Vocal attributes: age, sex, accent, acoustic features and bias (14 entries). |
| [`5-evaluacion-y-datos.md`](5-evaluacion-y-datos.md) | Metrics and datasets, including Spanish data (13 entries). |
| [`matriz-bancos-DASR.xlsx`](matriz-bancos-DASR.xlsx) | Matrix of 16 DASR criteria against 26 key works, with a codebook, an evidence sheet and 12 project decisions. |
| [`pdfs/`](pdfs/) | Local copies of 79 open-access papers; [`pdfs/INDICE.md`](pdfs/INDICE.md) maps each entry to its file and lists the 11 to download manually. |