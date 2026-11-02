# econ3916-lab11-sql

# SQL for Economists — Joins, Windows and the Cardinality Trap

## Objective

I wanted to see how the pandas operations I already know translate into SQL with DuckDB, and how a one-to-many join can quietly change every statistic computed after it.

## Methodology

- Built a customers table and a one-to-many orders table, then ran select, filter, sort, groupby and merge in both pandas and DuckDB SQL, and checked that both gave the same rows.
- Joined customers to orders with a LEFT JOIN, then ran the same Welch t-test on revenue (East vs. West) before and after the join.
- Used window functions (AVG OVER PARTITION BY and RANK) to put each region's average and each customer's rank on every row without collapsing the rows.
- Ran WHERE income != 50000 on a five-row survey table with two missing incomes, in both SQL and pandas.
- Queried a public Parquet file of about 3 million NYC taxi trips over HTTP with DuckDB, without downloading it, so that only the columns the query needed were read.
- Asked an AI to translate three of my pandas steps (groupby, merge and filter) into SQL, revised my prompt once, and compared the row counts of its queries with my pandas results.

## Key Findings

- **The cardinality trap.** A one-to-many LEFT JOIN turned 600 customer rows into 3,612 rows. The same Welch t-test on the same revenue figures went from t = 0.07 (p = 0.9413) to t = 0.71 (p = 0.4749). The repeated rows shrank the standard error and moved the difference, so the test acted as if it had far more customers than it did. Neither p-value is below 0.05 here, but the test no longer reflected the number of customers I had.
- **pandas and SQL do the same operations.** A groupby is a GROUP BY and a merge is a JOIN, and the two versions returned the same rows.
- **Window functions keep every row.** They gave me group averages and ranks without the collapsing that GROUP BY does.
- **SQL uses three-valued logic.** WHERE income != 50000 kept 3 of 5 survey rows, because a comparison with a missing income is unknown rather than true, and WHERE keeps only the rows where the condition is true. The same filter in pandas kept 5.
- **DuckDB can query a file it never downloads.** It read only the columns the query used from the 3 million taxi trips and returned a small summary table.
- **AI translation needs checking.** The AI's first Step 7 query dropped the missing-income rows without being told to. After I told it to keep them, the row counts of its three queries matched my pandas results. Matching row counts are a first check and do not prove the values are the same.
