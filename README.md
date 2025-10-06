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

## 💾 Dataset Details

The dataset is synthetically generated to model a common big data scenario. It's designed with specific columns that are used to test the effectiveness of filtering and aggregation performance after applying different Delta Lake optimizations.

### Row Structure
The base CSV file contains 10 million rows. In the notebook, this data is appended to the Delta table multiple times to create the final benchmark table, which contains **over 160 million rows**. Each row represents a single record containing a set of identifiers and associated numerical values. This large number of rows helps to simulate a production-scale dataset where performance optimizations become critically important.

### Column Descriptions
The table consists of nine columns, which can be grouped into identifiers and value-based metrics. The columns **`id1`** and **`id2`** are the most important, as they are used directly in the benchmark queries and as the target columns for the **Z-Ordering** and **Liquid Clustering** optimizations.

| Column | Data Type | Description | Role in Project |
| :--- | :--- | :--- | :--- |
| **`id1`** | String | A categorical identifier with a fixed number of unique values. | **Primary column** for filtering, aggregation, and clustering. |
| **`id2`** | String | Another categorical identifier. | **Secondary column** for filtering, aggregation, and clustering. |
| **`id3`** | String | A unique identifier for each record. | General ID, not used in optimizations. |
| **`id4`** | Integer | A numerical identifier or attribute. | Supplementary data. |
| **`id5`** | Integer | A numerical identifier or attribute. | Supplementary data. |
| **`id6`** | Integer | A numerical identifier or attribute. | Supplementary data. |
| **`v1`** | Integer | A numerical metric or value. | Supplementary data. |
| **`v2`** | Integer | A numerical metric or value. | Supplementary data. |
| **`v3`** | Double | A floating-point metric or value. | Supplementary data. |

***

## 👥 Collaborators

This project is a collaborative effort by a dedicated team of 5

- [Shivansh](https://github.com/sh1vanshgaur)
- [Pulkit](https://github.com/pulkitjn3010)
- [Anuj](https://github.com/animus08)
- [Nishtha](https://github.com/novnishtha)
- [Riya](https://github.com/Riyag012)