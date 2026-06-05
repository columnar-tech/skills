---
name: databow
description: Query SQL databases with the databow CLI. Use when the user wants to run SQL queries against any database with an ADBC driver (BigQuery, ClickHouse, Databricks, DataFusion, DuckDB, Exasol, Microsoft SQL Server, MySQL, PostgreSQL, Redshift, SingleStore, Snowflake, Spark, SQLite, Trino, etc.), export query results to JSON/CSV/Arrow IPC files, or work with ADBC connection profiles.
license: Apache-2.0
metadata:
  author: Columnar
  version: "0.1.0"
---

# databow

databow is a command-line tool for querying databases via [ADBC](https://arrow.apache.org/adbc/current/index.html) (Arrow Database Connectivity). It exposes an interactive SQL shell as well as a non-interactive mode that is well-suited for use by agents and scripts.

## Prerequisites

### Install databow

databow must be installed. Check with `databow --help`.

Common install methods:

```bash
uv tool install databow
# or
pipx install databow
# or
cargo install databow
```

### Install ADBC driver

An ADBC driver for the target database must be installed. The recommended way is via [dbc](https://docs.columnar.tech/dbc/llms.txt), which installs drivers in a location databow discovers automatically:

```bash
dbc install duckdb
dbc install postgresql
dbc install snowflake
# ...etc.
```

If a query fails with a driver-not-found error, install the appropriate driver with `dbc install <driver>` before retrying.

## Connecting to a database

There are two ways to specify connection details. Use whichever is more appropriate for the task.

### Method 1: CLI flags

Specify the driver and connection options on the command line:

```bash
databow --driver duckdb --uri warehouse.db
databow --driver postgresql --uri postgresql://user:pwd@localhost:5432/db
databow --driver mysql --uri 'root:pwd@tcp(localhost:3306)/sys'
```

Available flags:

- `--driver <name>` — ADBC driver name (e.g. `duckdb`, `postgresql`, `mysql`, `sqlite`, `clickhouse`, `snowflake`, `bigquery`, `flightsql`, `mssql`, `redshift`, `databricks`, `trino`).
- `--uri <uri>` — Database URI (driver-specific format).
- `--username <user>` — Database username.
- `--password <pwd>` — Database password.
- `--option key=value` — Driver-specific option. May be repeated.

### Method 2: Connection profiles

[ADBC connection profiles](https://arrow.apache.org/adbc/current/format/connection_profiles.html) are TOML files that store driver and connection options, e.g. `~/.config/adbc/profiles/warehouse.toml`:

```toml
profile_version = 1
driver = "clickhouse"

[Options]
uri = "http://localhost:8123"
username = "user"
password = "pass"
```

Use the profile by name (if it lives in a profile search location) or by path:

```bash
databow --profile warehouse
databow --profile /path/to/warehouse.toml
```

Prefer profiles when the same connection will be used repeatedly, or when credentials should not appear on the command line.

## Common workflows

### Quick ad-hoc query against in-memory DuckDB

```bash
databow --driver duckdb --query "SELECT 1 + 1 AS two"
```

### Query a PostgreSQL database and export to CSV

```bash
databow \
  --driver postgresql \
  --uri postgresql://user:pwd@localhost:5432/app \
  --query "SELECT id, email, created_at FROM users WHERE created_at > now() - interval '7 days'" \
  --output recent_users.csv
```

### Run a SQL file using a stored profile and write CSV

```bash
databow --profile warehouse --file reports/weekly.sql --output weekly.csv
```

### Produce a Markdown table for inclusion in a report

```bash
databow --driver duckdb --mode ascii-markdown \
  --query "SELECT * FROM 'https://blobs.duckdb.org/data/penguins.csv'" \
  > penguins.md
```

## Troubleshooting

- **Driver not found / failed to load.** Install the driver with `dbc install <driver>` (e.g. `dbc install duckdb`). Verify the driver name passed to `--driver` matches.
- **Authentication errors.** Double-check `--uri`, `--username`, `--password`, and any driver-specific `--option` values. For databases that require certificates, tokens, or extra parameters, see the upstream databases reference.
- **Profile not found.** Either pass an explicit path with `--profile /path/to/profile.toml`, or place the profile in an [ADBC profile search location](https://arrow.apache.org/adbc/current/format/connection_profiles.html#profile-search-locations).
- **Garbled or wrapped output.** Use `--output result.csv` (or another file format) instead of stdout, or switch `--mode` to `nothing` / `ascii-no-borders` for plain rows.
- **Hanging command.** If `databow` was invoked without `--query`, `--file`, or stdin input, it has dropped into the interactive REPL. Re-invoke with one of those flags for non-interactive use.
