# Query Store Plan Regression Detection (SQL Server)


This repository contains a T-SQL script that identifies query performance regressions using SQL Server Query Store. It compares recent execution performance against a baseline window to detect slowdowns at the query + plan level.

**Overview**

The script analyzes runtime statistics from:
sys.query_store_runtime_stats
sys.query_store_plan
sys.query_store_query

It builds two time windows:
Baseline period: last 7 days excluding the most recent 1 day
Current period: last 1 day

It then compares average execution duration and flags queries that have significantly degraded.

**What It Detects**

The query identifies cases where:
Execution time has increased by 50% or more compared to baseline
The query has executed more than 20 times in the current window
The total current execution time exceeds 100,000 microseconds

This helps isolate meaningful regressions and avoids noise from infrequent queries.

**Output Columns**

Column	Description
query_id	Unique Query Store query identifier
plan_id	Execution plan identifier
baseline_duration	Average duration in baseline period
current_duration	Average duration in current period
slowdown_factor	Ratio of current vs baseline performance
baseline_exec_count	Number of executions in baseline window
current_exec_count	Number of executions in current window
baseline_total_duration	Total runtime in baseline period
current_total_duration	Total runtime in current period
Logic Summary

The script uses weighted averages:
avg_duration = SUM(avg_duration * executions) / SUM(executions)

This ensures that high-frequency executions have appropriate impact on performance calculations.

Two CTEs are used:
baseline - historical performance reference
current - recent performance snapshot

Then a join compares the same query + plan across both periods.

**How to Use**

Enable Query Store on the database:
ALTER DATABASE YourDB SET QUERY_STORE = ON;
Run the script in SQL Server Management Studio or Azure Data Studio.
Review results ordered by slowdown_factor DESC.

**Use Cases**

Detecting plan regressions after deployments
Identifying performance degradation due to parameter sniffing
Monitoring unexpected workload slowdowns
Supporting performance RCA (root cause analysis)

**Notes**

Requires Query Store to be enabled and populated
Time windows can be adjusted depending on workload volatility
Thresholds (1.5x slowdown, 20 executions, 100,000 duration) should be tuned per environment

**Example Interpretation**

If a query shows:
baseline_duration = 10ms
current_duration = 40ms
slowdown_factor = 4.0
This indicates a 4x regression, likely due to plan change or data skew.

**License**

Use freely for monitoring and performance diagnostics in SQL Server environments.
