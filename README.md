# Benchmarking Delta Lake Optimization Techniques in Spark

This project demonstrates and benchmarks various optimization techniques available in Delta Lake to improve query performance on large datasets using PySpark. The primary methods explored are **file compaction**, **Z-Ordering**, and **Liquid Clustering**.

***

## 📖 Overview

The core of this project is the `delta-optimizations.ipynb` Jupyter Notebook. It performs the following steps:
1.  **Initial Data Ingestion**: A 3.71 GB dataset with over 160 million rows is created by ingesting a base CSV file and appending it multiple times to simulate a scenario with many small files.
2.  **Applying Optimizations**: Several versions of the Delta table are created, each with a different optimization strategy applied:
    * **Normal**: The initial unoptimized table with numerous small files.
    * **Compaction**: Small files are compacted into larger, optimally sized files using the `OPTIMIZE` command.
    * **Z-Ordering**: Data is co-located based on specified columns (`id1` and `id2`) to enable efficient data skipping.
    * **Liquid Clustering**: A modern, flexible alternative to Z-Ordering for data layout optimization.
3.  **Benchmarking**: Three distinct queries are run against each version of the table to measure and compare the performance improvements.

***

## 📁 Project Structure

The project requires the following structure.  
The notebook will generate the `delta-test`, `liquid-test`, and `spark-warehouse` directories.

```
.
├── data/
│ └── data-1g/ <-- Required: Place Parquet files here for Liquid Clustering
├── notebooks/
│ └── delta-optimizations.ipynb <-- The main Jupyter Notebook
| └── mergeOperation.ipynb
├── raw_data/
│ └── N_1e7_K_1e2_single.csv <-- Required: Source CSV file
└── spark-warehouse/ <-- Generated: Default Spark warehouse

```
***

## 📊 Benchmark Results

The performance of each optimization technique was measured against three query types:
* **Query A**: Filters and aggregates on `id1`.
* **Query B**: Filters and aggregates on `id2`.
* **Query C**: Filters and aggregates on both `id1` and `id2`.

The results clearly show the impact of different data layout strategies on query performance.

| Version / Optimization | Query A (id1) [sec] | Query B (id2) [sec] | Query C (id1 & id2) [sec] | Implementation Time | Files Skipped |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Normal (v3)** | 12.80 | 6.90 | 6.17 | – | 0 / 32 |
| **Optimize / Compaction (v4)** | 11.40 | 4.80 | 5.72 | 7m 36.8s | 0 / 4 |
| **Z-Order on `id1` (v5)** | **7.20** | 12.70 | 2.17 | 11m 51.3s | **2 / 3** |
| **Z-Order on `id1` & `id2` (v6)** | **4.15** | 6.30 | **1.28** | 15m 54.7s | **1 / 3** |
| **Liquid Clustering (id1)** | 13.80 | 7.80 | 4.60 | 5m 34.8s | ? |

### Key Findings
* **Compaction** provides a baseline improvement by reducing the overhead of reading many small files.
* **Z-Ordering on `id1`** dramatically speeds up queries filtering on `id1` but negatively impacts queries on `id2`, as the data layout is now biased.
* **Z-Ordering on both `id1` and `id2`** offers the best overall performance, especially for queries that filter on both columns, showcasing its power in multi-dimensional data clustering.

***

## 🚀 How to Run

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/sh1vanshgaur/delta-optimizations
    ```

2.  **Set up the environment:**
    Ensure you have a Python environment with the required libraries installed.
    ```bash
    pip install pyspark delta-spark jupyter
    ```

3.  **Prepare Data Sources:**
    First, create the necessary directories. Then, manually download the data files from the provided links and place them in the correct locations.

    * **Create directories:**
        ```bash
        mkdir -p raw_data data/data-1g
        ```
    * **Download the source CSV file:**
        * **Link:** `https://drive.google.com/drive/folders/1T22yWOrSXaDnRO2g6MnGje4nYBSvg4fG?usp=sharing`
        * **Action:** Place this file inside the `raw_data/` directory and name it `N_1e7_K_1e2_single.csv`.

    * **Download the Parquet files** :
        * **Link:** `https://drive.google.com/drive/folders/1T22yWOrSXaDnRO2g6MnGje4nYBSvg4fG?usp=sharing`
        * **Action:** Download the zip file, extract its contents, and place the resulting Parquet files inside the `data/data-1g/` directory.

4.  **Execute the Notebook:**
    Launch Jupyter Notebook and run the cells in `notebooks/delta-optimizations.ipynb` to reproduce the results.

# H2O.ai Group-By Benchmark Dataset

## 📝 Overview

This repository contains a sample dataset from **H2O.ai's benchmark suite**, designed specifically to test the performance and speed of data processing systems and machine learning algorithms.

The primary purpose of this dataset is not to model a real-world scenario but to provide a standardized, large-scale dataset to measure the efficiency of common data manipulation tasks, particularly **group-by operations** followed by **aggregations**.

---

## Dataset Structure

The dataset consists of several identifier columns (used for grouping) and value columns (used for calculations).

### Column Explanations

The columns fall into two main categories:

1.  **Identifier Columns (`id1` - `id6`):** These are the grouping keys. They are categorical IDs with varying numbers of unique values (cardinality) and data types, designed to test how a system handles different grouping scenarios.
2.  **Value Columns (`v1` - `v3`):** These are the numerical columns on which aggregate functions are performed.

| Column Name | Data Type | Description |
| :---------- | :---------- | :---------- |
| `id1` | String | Categorical ID, low cardinality (e.g., 100 unique values). |
| `id2` | String | Categorical ID, medium cardinality. |
| `id3` | String | Categorical ID, high cardinality. |
| `id4` | Integer | Categorical ID, integer type. |
| `id5` | Integer | Categorical ID, integer type. |
| `id6` | Integer | Categorical ID, integer type. |
| `v1` | Integer | Integer value for aggregation (e.g., `sum()`, `count()`). |
| `v2` | Integer | Integer value for aggregation. |
| `v3` | Float | Floating-point value for aggregation (e.g., `mean()`, `stddev()`). |

## 👥 Collaborators

This project is a collaborative effort by a dedicated team of 5

- [Shivansh](https://github.com/sh1vanshgaur)
- [Pulkit](https://github.com/pulkitjn3010)
- [Anuj](https://github.com/animus08)
- [Nishtha](https://github.com/novnishtha)
- [Riya](https://github.com/Riyag012)