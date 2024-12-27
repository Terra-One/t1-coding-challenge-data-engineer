# Data Engineering Challenge

## 1. Coding Challenge

### 1.1 Introduction

In this challenge, you’ll work with a locally provided CSV file that simulates one month of day-ahead energy market bids. The data is intentionally messy, containing various subtle errors and inconsistencies. Your task is to approach this as a data engineer: load the data, explore it, understand it, clean and harmonize it, and ultimately perform some simple calculations — entirely using a local environment. No external APIs, secret tokens, or remote services are involved. The goal is not just to run a fixed script but to get a real sense of the data’s condition.

We understand that you may use tools like ChatGPT or other LLMs to assist in solving this case study, which is perfectly fine, but we encourage you to first make your own assumptions and ensure that you fully understand the concepts you're defining when presenting your solutions.

### 1.2 Data Overview

The provided CSV file, `energy_data.csv`, contains about one month of data, with roughly 5000 bids per day. Each row is supposed to represent an energy market bid. The fields include:

- **Timestamp**: In ISO format. 
- **Price**: Bid price in €/MWh 
- **Volume**: Bid volume in MWh
- **Sell_Buy**: Indicates if the bid is a Sell or Buy order.

**Note:** The dataset may contain data quality issues. It’s up to you to discover and address them.

### 1.3 Goal

1. **Data Exploration & Understanding:**  
   Load the CSV, inspect the data, and note the types of issues present. This might mean reading a small sample in a Python shell or Jupyter Notebook before integrating into an Airflow pipeline.

2. **Data Cleaning & Normalization:**  
   Harmonize the data by ensuring consistency in timezones, numeric fields, and categorical values. Address any data quality issues encountered.

3. **Data Quality Checks:**  
   Make sure that for each day you have a reasonable spread of bids. Consider what to do about extreme outliers or missing time intervals.

4. **Partitioning & Intermediate Storage:**  
   Split the cleaned data by day, saving each day’s cleaned subset as a Parquet file in a `processed_data/` folder. This simulates a pipeline step where raw data is normalized and stored for downstream use.

5. **Aggregation & Scenario Simulation:**

    For each daily Parquet file:

   1. **Hourly Aggregates**  
        - Compute hourly aggregates for **sell** orders: average sell price, min sell price, max sell price, total sell volume.  
        - Compute hourly aggregates for **buy** orders: average buy price, min buy price, max buy price, total buy volume.
        - Calculate **Volume-Weighted Average Price (VWAP)** for both all buy and separately all sell trades using:
        $$
        \text{VWAP} = \frac{\sum (\text{Price}_i \times \text{Volume}_i)}{\sum \text{Volume}_i}
        $$

   2. **Supply–Demand Equilibrium Calculation**  
      - Sort **sell** orders by ascending price to form a simplistic “supply curve.”
      - Sort **buy** orders by descending price to form a simplistic “demand curve.”
      - Identify an **intersection** where the cumulative supply volume meets or exceeds the total buy volume (demand). The price at that point is our “equilibrium.”

   3. **Renewables Shift Simulation** (±5000 MWh) 

      Let's assume that we want to simulate the impact of additional wind or solar generation, which increases the total energy supply by 5000 MWh (e.g., “+5000” scenario). Conversely, in other hours, there is a shortage in renewables (e.g., lower wind speeds) that effectively reduces the total supply by 5000 MWh (“–5000” scenario).

      - **Objective:** We model this by **shifting** the total **supply** (or “sell” volume) by ±5000 MWh.  
        - **Step 1:** For each hour, sum the total sell volume.  
        - **Step 2:** Add 5000 MWh from the total “sell” side, distributing that shift proportionally across each sell trade in that hour.  
        - **Step 3:** Re-run the intersection logic with the new “shifted” volumes to see how the final equilibrium price changes.  
        - **Step 4:** Repeat for –5000 MWh.
      - **Outcome:** Compare the “shifted” equilibrium price (±5000 MWh renewables) to the original scenario. This gives a rough sense of how having more or fewer renewables might move the price.

6. **Final Summary Output**  
   Generate a `hourly_summary.csv` that, for each hour, shows:
   - Hourly metrics (average, min, max, total volume, VWAP)
   - The original equilibrium price
   - The equilibrium price under +5000 MWh renewables
   - The equilibrium price under –5000 MWh renewables

### 1.4 Requirements

- **Local Environment Only:** No external APIs or credentials should be required.
- **Packages:**  
  - **Airflow** for orchestrating tasks.
  - **pandas** for data loading, cleaning, and transformations.
  - **pytz** or Python’s built-in timezone support for handling timezones.
  - **numpy** optional, but helpful for numeric operations.
  
### 1.5 Steps Outline

1. **Setup Airflow & Dependencies**  
   - Configure a local Airflow environment and create a DAG that orchestrates the tasks.

2. **Data Loading & Initial Inspection**  
   - Use a PythonOperator (or similar) to read the CSV file.

3. **Data Cleaning & Normalization**  
   - Standardize timestamps and data formats.
   - Normalize numerical and categorical fields.
   - Handle missing or irregular values.

4. **Partitioning & Storage**  
   - Split data by day after cleaning.
   - Save each partition as a Parquet file.

5. **Aggregation & Scenario Simulation**  
   - For each partition, compute desired hourly metrics
   - Implement a simple scenario analysis where you introduce a demand shift (e.g., ±5000 MWh) and observe the impact on a chosen pricing metric (e.g., median price).

6. **Final Summary**  
   - Combine all hourly results for each day in the dataset into one summary CSV (e.g., `hourly_summary.csv`) that reflects both the original and shifted-demand outcomes.

### 1.6 Expected Outcome

- A functioning Airflow DAG that:
  - Reads and processes the CSV data.
  - Cleans and normalizes the dataset.
  - Partitions and stores the data in Parquet format.
  - Computes and logs relevant metrics, including scenario simulations.
  - Consolidates final results into a single summary output (e.g., `hourly_summary.csv`).

## 2. Conceptual Task: Scaling & Performance Strategy

**Task:**  
Draft a **short strategy document** (around half a page) explaining how you would **scale the pipeline** if:
1. Data volumes grow significantly (e.g., more markets and higher-frequency records).
2. The team needs to maintain or improve query performance on the final data stored in a time series database and pipeline reliability.

**Focus Areas:**  
- **Scaling Approaches**  
- **Monitoring & Alerts**  
- **Caching & Partitioning**  

**Deliverable:**  
A brief outline (bullet points or short paragraphs) describing the **main steps** you would take to handle future expansions—how to measure performance, respond to pipeline slowdowns, and keep data queries efficient as volumes grow. 
