# Stylometric Analysis of 19th-Century French Literature with PySpark

A distributed NLP pipeline built with **PySpark** and **Spark NLP** to compare the writing style of 19th-century French novels across literary movements (romanticism, realism…), with a strong focus on **Spark performance tuning**.

## What it does

1. **Downloads** a corpus of public-domain French novels with their metadata (author, literary movement).
2. **Annotates** the texts with Spark NLP: sentence detection, tokenization, normalization and lemmatization.
3. Computes two **readability indices** for each book:
   - **Kandel–Moles** (adapted to French)
   - **Flesch–Kincaid** (designed for English)
4. Computes the Kandel–Moles index on **sliding windows** of 100 and 200 paragraphs, to see how difficulty changes within a book.
5. Checks **Zipf's law** (word frequency vs rank on a log-log scale) for each book.

## Key findings

- **13 of the 15 easiest books** to read are by romantic authors, while the two hardest are realist novels.
- Flesch–Kincaid systematically rates French texts as harder than Kandel–Moles does, because it is calibrated for English and French words have more syllables on average.
- Readability varies a lot *within* a book: *Rome* (Zola), the hardest book overall (58.3), has windows ranging from 50 to 71.8.
- All books closely follow Zipf's law.

## Spark performance work

Jobs, stages and shuffles were analyzed with the **Spark UI** (Jobs, Stages, SQL and Storage tabs), then optimized:

- **Broadcast joins** for the small metadata table, which removes a shuffle
- **Repartitioning** on the grouping key (`repartition(16, "id")`), immediately followed by **caching** to avoid recomputing the annotation step
- **Early column selection and filtering** to reduce shuffle size before wide transformations
- **Kryo serialization** and larger driver/executor memory (1 GB → 8 GB) to cache intermediate results without out-of-memory errors
- **Storage format comparison:** CSV cannot store nested arrays; **Parquet is 2.3× smaller than JSON** (91 MB vs 213 MB) and faster to read

## Tech stack

**Python** · PySpark · Spark NLP · Spark SQL (window functions, higher-order functions) · pandas · Plotly · **Quarto**

## How to run

Requirements: Python 3, Java 8/11/17 (needed by Spark) and Quarto.

```bash
pip install pyspark spark-nlp pandas plotly
quarto render report.qmd
```

## Authors

Group project — Big Data course, Master in Mathematics & Computer Science for Data Science, Université Paris Cité.

Steven Lahoud, Anthony Fernandez
