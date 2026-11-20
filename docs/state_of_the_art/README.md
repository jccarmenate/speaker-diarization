# State of the Art

This directory contains the systematic literature review that supports the design decisions of the DASR pipeline.

## Contents

| File | Description |
|---|---|
| `state_of_the_art.pdf` | Literature review covering ASR, speaker diarization, and vocal attribute analysis. |
| `state_of_the_art.tex` | LaTeX source of the literature review. The compiled PDF is `state_of_the_art.pdf`. |
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