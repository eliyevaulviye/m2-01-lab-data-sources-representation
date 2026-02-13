![logo_ironhack_blue 7](https://user-images.githubusercontent.com/23629340/40541063-a07a0a8a-601a-11e8-91b5-2f13e4e6b441.png)

# Lab | Data Sources and Representation

## Overview

This lab gives you hands-on practice with the core concepts from the lesson on data sources, structure, and representation. You will work with multiple real-world-style datasets in different formats, explore how the same information can be stored differently, compare row-major and column-major layouts empirically, and reason about the trade-offs of each representation choice. The emphasis is on building practical intuition: by the time you finish, you should be able to look at a dataset and articulate why it is stored the way it is, what queries will be fast or slow, and what you would change for a different use case.

## Learning Goals

By the end of this lab, you should be able to:

- Identify the source type of a dataset and describe its characteristics (user input, system-generated, internal, third-party)
- Work with data in multiple formats (CSV, JSON, Parquet) and convert between them
- Observe and explain the performance differences between row-major and column-major data access patterns
- Compare text and binary format sizes empirically
- Reason about the trade-offs between structured and unstructured representations
- Choose an appropriate representation for a given analytical task and justify your choice

## Setup and Context

You will work in a Jupyter Notebook, building small datasets in memory and loading prepared files. No external APIs or large downloads are required. The focus is on reasoning about format and structure choices through direct experimentation.

## Requirements

- Fork this repository to your own GitHub account.
- Clone your fork to your machine.
- Make sure you have the following Python packages installed: `pandas`, `numpy`, `pyarrow` (for Parquet support), and `json` (standard library).
- You can install the required packages with:

```bash
pip install pandas numpy pyarrow
```

## Getting Started

Create a new notebook and name it `m2-01-data-sources-representation-lab.ipynb`. Complete all tasks in this notebook. Use markdown cells to explain your reasoning where requested. Before you submit, restart your kernel and run the notebook top to bottom to make sure it executes cleanly.

## Tasks

### Task 1: Identify and Characterize Data Sources

In this task, you will examine three different datasets and classify each one by its source type.

**Step 1:** Create the following three datasets in memory as Python data structures (lists of dictionaries):

**Dataset A** — A list of 8 user registration entries with fields: `username`, `email`, `age`, and `signup_date`. Include at least 2 entries with problems: one with an invalid email (missing `@`), one with age as a string instead of a number, and one with an empty username.

**Dataset B** — A list of 10 application log entries with fields: `timestamp`, `service`, `level` (INFO/WARNING/ERROR), and `message`. Include entries from at least 3 different services.

**Dataset C** — A list of 6 product inventory records with fields: `sku`, `product_name`, `quantity_in_stock`, `warehouse_location`, and `last_updated`.

**Step 2:** For each dataset, write a short markdown paragraph answering:
- What type of data source does this represent (user input, system-generated, internal database)?
- What are the likely quality issues you would expect from this source in production?
- What kind of validation would you apply before using this data in an ML pipeline?

**Step 3:** Write a validation function called `validate_registrations(entries)` that takes Dataset A and returns two lists: `valid_entries` and `invalid_entries`. A valid entry must have a non-empty username, an email containing `@`, and an age that can be converted to a positive integer. Test your function on Dataset A and print the counts.

### Task 2: Work with Multiple Data Formats

This task explores how the same data looks and behaves in different formats.

**Step 1:** Create a pandas DataFrame called `rides` with 500 rows and the following columns:
- `ride_id`: integers from 1 to 500
- `driver_id`: random integers between 1000 and 1050
- `distance_km`: random floats between 1.0 and 30.0
- `fare_usd`: random floats between 5.0 and 75.0
- `ride_type`: randomly chosen from `["standard", "premium", "pool"]`
- `city`: randomly chosen from `["Berlin", "Seoul", "Nairobi", "Toronto", "Lima"]`
- `timestamp`: a date range starting from `"2025-01-01"` with minute frequency

**Step 2:** Save the DataFrame in three formats:
- CSV: `rides.csv`
- JSON: `rides.json` (using `orient="records"` and `lines=True`)
- Parquet: `rides.parquet`

**Step 3:** Compare the file sizes of all three formats. Print the size of each file in KB and compute how many times smaller Parquet is compared to CSV.

**Step 4:** Read each file back into a new DataFrame and verify that the data is identical by comparing shapes and a sample of values. Note any differences you observe (for example, type changes after reading from CSV).

**Step 5:** Write a markdown cell summarizing: Which format preserved types most faithfully? Which was smallest? When would you choose each format?

### Task 3: Row-Major vs Column-Major Access Patterns

This task lets you observe the performance differences between row-level and column-level data access empirically.

**Step 1:** Create a large DataFrame with 200,000 rows and 20 numeric columns (random float values). Name the columns `feature_0` through `feature_19`.

**Step 2:** Measure and compare the time for these two operations:

**Operation A — Column access:** Compute the mean of `feature_0` using the pandas column accessor (`df["feature_0"].mean()`).

**Operation B — Row iteration:** Loop through the first 10,000 rows using `df.iloc[i]` and sum the values of `feature_0` manually.

Use `%%timeit` or `%time` to measure each. Record the results.

**Step 3:** Now convert the DataFrame to a NumPy array and repeat both operations:

**Operation C — NumPy column access:** Compute the mean of column 0 (`arr[:, 0].mean()`).

**Operation D — NumPy row access:** Access the first 10,000 rows sequentially (`arr[i]` in a loop) and sum column 0.

Record the results.

**Step 4:** Write a markdown cell that answers:
- Why is column access in pandas so much faster than row iteration?
- Why does the gap narrow (or change) when using NumPy?
- What does this tell you about how pandas and NumPy lay out data in memory?

### Task 4: Text vs Binary Format Deep Dive

This task explores the text-versus-binary trade-off with a focus on practical consequences.

**Step 1:** Create a DataFrame with 100,000 rows and 5 numeric columns of random float values (use `np.random.seed(42)` for reproducibility). Add one string column called `category` with values randomly chosen from `["alpha", "beta", "gamma"]`.

**Step 2:** Save as CSV and Parquet. Compare file sizes.

**Step 3:** Read the CSV back and check the dtypes. Then read the Parquet back and check the dtypes. Write a markdown cell explaining any differences you observe in how types were preserved.

**Step 4:** Perform a timing comparison: measure how long it takes to read the full dataset from CSV versus from Parquet. Record both times.

**Step 5:** Parquet supports reading only specific columns without loading the entire file. Read only the `category` column from the Parquet file and time it. Compare to reading the full file. Write a markdown cell explaining why this selective read is possible with Parquet but not with CSV.

### Task 5: Representation Choices and Trade-Offs

This capstone task asks you to think critically about how representation decisions affect analysis.

**Step 1:** Create a flat (denormalized) DataFrame representing an online bookstore with at least 12 rows. Include columns: `title`, `author`, `format` (Paperback/E-book), `publisher_name`, `publisher_country`, `price`. Make sure at least two books have multiple formats (so the publisher info is repeated).

**Step 2:** Create a normalized version of the same data using two DataFrames:
- `books` with columns: `title`, `author`, `format`, `publisher_id`, `price`
- `publishers` with columns: `publisher_id`, `publisher_name`, `publisher_country`

Verify your normalization by joining the two tables back together and comparing with the flat version.

**Step 3:** Create a document-oriented version of the same data as a list of dictionaries (JSON-style), where each book is a document containing nested publisher information and a list of editions.

**Step 4:** For each representation (flat, normalized, document), write and execute code to answer these three questions:
1. What are all books by a specific author?
2. What is the average price across all editions?
3. Update a publisher's country (simulate the publisher moving to a new country). How many records/documents need to change?

**Step 5:** Write a summary markdown cell comparing the three representations. For each question above, which representation made the answer easiest? Which representation would you choose if the publisher information changes frequently? Which would you choose if each book is always accessed as a standalone record?

## Submission

### What to submit

Submit the following file:

- A notebook file `m2-01-data-sources-representation-lab.ipynb` containing all five tasks with code and markdown explanations

### Definition of done (checklist)

Before you submit, make sure:

- [ ] The notebook runs **top to bottom** without errors after a kernel restart.
- [ ] Task 1 includes all three datasets, source type classification, and a working validation function.
- [ ] Task 2 saves data in three formats, compares sizes, and discusses type preservation.
- [ ] Task 3 includes timing measurements and a clear explanation of row-major vs column-major behavior.
- [ ] Task 4 compares CSV and Parquet read performance and demonstrates selective column reads.
- [ ] Task 5 includes all three representations with code answering the three analytical questions and a comparison summary.
- [ ] Markdown cells provide clear reasoning (not just code output).
- [ ] The notebook file is saved and included in your git commit.

### How to submit (Git workflow)

When you are done, make sure all changes are saved, then run:

```bash
git add .
git commit -m "Solved m2-01 lab"
git push -u origin HEAD
```

- Make a pull request.
- Paste the link to your pull request in the Student Portal.

## Evaluation Criteria

Your work will be evaluated on correctness, analytical reasoning, and completeness. Correctness means your code runs cleanly and produces consistent results. Analytical reasoning means your markdown explanations demonstrate genuine understanding of why formats and layouts behave the way they do, not just what happens. Completeness means every task and subtask is addressed with both code and written analysis. The goal is to demonstrate that you can make informed decisions about data representation in real-world scenarios.
