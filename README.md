# NYC Taxi Analysis

A PySpark analysis of NYC yellow taxi trip data on Databricks, covering exploratory analysis,
partitioned storage strategies, partition-pruning benchmarks, and Structured Streaming.

## What's in the notebook

The main asset is [`nyc-taxi-analysis.ipynb`](nyc-taxi-analysis.ipynb), which walks through:

1. **Load data** — reads the `samples.nyctaxi.trips` sample table (~22K trips)
2. **Explore** — schema inspection and summary statistics for fares and distances
3. **Revenue aggregation** — top 10 pickup ZIP codes by total revenue (zero-distance and
   non-positive-fare trips excluded)
4. **Partitioned writes** — writes the same data twice: unpartitioned flat Parquet and
   date-partitioned Parquet (`pickup_date=YYYY-MM-DD/` folder layout) to a Unity Catalog volume
5. **Partition-pruning benchmark** — times the same date-filtered query against both layouts
6. **Structured Streaming** — two streaming demos that treat the Parquet drops as live data,
   aggregating per-ZIP trip counts into an in-memory table

## Key findings

- Total trips: ~21,932 (mean fare $12.35, mean distance 2.85 miles)
- Highest-revenue pickup ZIP: **11422** (~$19.0K), followed by 11371 and 10001
- On this small dataset the partitioned read is **not faster** than the flat scan — partition
  pruning pays off at scale, and file-listing overhead dominates when data is tiny

## Requirements

- A Databricks workspace with **serverless compute** (the notebook is written for serverless;
  no cluster configuration needed)
- Access to the `samples.nyctaxi.trips` sample dataset
- A writable Unity Catalog volume — the notebook writes to
  `/Volumes/workspace/default/raw_data/nyc_taxi/` (adjust the paths if your volume differs)

## How to run

Open the notebook in a Databricks workspace, attach serverless compute, and **Run All**.
All writes go to the volume paths above; the streaming cells use `trigger(availableNow=True)`
so each stream processes the available files and completes on its own — no need to stop it
manually.

## Notes

- The notebook runs on Databricks serverless, where direct `s3a://` (DBFS/Hadoop) paths are
  unavailable: all reads/writes use Unity Catalog volumes, and streaming requires an explicit
  `checkpointLocation`.
- Data spans pickup dates 2016-01-01 to 2016-02-29.
