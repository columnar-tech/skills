---
name: duckdb-adbc
description: Query external databases from DuckDB using the ADBC (Arrow Database Connectivity) community extension. Use when the user wants to run SQL against any database with an ADBC driver (BigQuery, ClickHouse, Databricks, PostgreSQL, MySQL, Redshift, SingleStore, Snowflake, SQL Server, SQLite, Trino, etc.) from within DuckDB, attach an external database as a DuckDB catalog, read remote data with `read_adbc`, execute arbitrary remote DDL/DML with `adbc_execute`, or manage ADBC connection profiles/drivers via `dbc`.
license: Apache-2.0
metadata:
  author: Columnar
  version: "0.1.0"
---

# duckdb-adbc

The `adbc` extension lets DuckDB (v1.4.5+ or v1.5.4+) query any database that has an [ADBC](https://arrow.apache.org/adbc/current/index.html) (Arrow Database Connectivity) driver — Snowflake, Databricks, BigQuery, PostgreSQL, MySQL, and more — directly from SQL, using zero-copy Arrow data transfer instead of legacy row-based APIs like ODBC/JDBC.

Repo: https://github.com/columnar-tech/duckdb-adbc-client

## Prerequisites

### 1. Install the extension

From within a DuckDB session (or a `.sql` file / `-c` invocation):

```sql
INSTALL adbc FROM community;
LOAD adbc;
```

If DuckDB reports it can't find the extension, confirm the DuckDB version is 1.4.5+ (or 1.5.4+) — the extension requires a recent build.

### 2. Install `dbc`

ADBC drivers themselves are installed and managed with `dbc`, a small CLI from Columnar. Check whether it's already available with `dbc --help`; if not, install it with whichever of these fits the environment:

```bash
# shell (macOS/Linux)
curl -LsSf https://dbc.columnar.tech/install.sh | sh

# Homebrew
brew install columnar-tech/tap/dbc

# uv
uv tool install dbc

# pipx
pipx install dbc

# PowerShell (Windows)
powershell -ExecutionPolicy ByPass -c irm https://dbc.columnar.tech/install.ps1 | iex

# winget (Windows)
winget install dbc
```

### 3. Install an ADBC driver for the target database

```bash
dbc install sqlite
dbc install postgresql
dbc install snowflake
# ...etc.
```

If a query fails with a driver-not-found error, install the appropriate driver with `dbc install <system>` before retrying.

## Connecting to a database

The extension connects via **ADBC connection profiles** — TOML files describing a driver and its options — referenced with a `profile://` URI.

### Create a profile

```toml
# mydb.toml
profile_version = 1
driver = "sqlite"

[Options]
uri = "./games.sqlite"
```

For remote/authenticated systems, add the relevant options, e.g.:

```toml
profile_version = 1
driver = "postgresql"

[Options]
uri = "postgresql://user:pwd@localhost:5432/app"
```

### Save the profile to a discoverable location

```bash
# Linux
mv mydb.toml ~/.config/adbc/profiles/

# macOS
mv mydb.toml ~/Library/Application Support/ADBC/Profiles/

# Windows
move "mydb.toml" "%LOCALAPPDATA%\ADBC\Profiles\"
```

Once saved, reference the profile as `profile://mydb` (filename minus `.toml`) from SQL, or point directly at a path.

Prefer profiles over embedding credentials in ad-hoc SQL — they keep secrets out of query text and can be reused across sessions.

## Common workflows

### Ad-hoc read via `read_adbc`

Best for one-off queries, or when pushing a projection/predicate down to the remote system (the extension does not automatically push down projections/predicates for attached tables — use `read_adbc` directly for that).

```sql
INSTALL adbc FROM community;
LOAD adbc;

SELECT * FROM read_adbc('profile://mydb', 'SELECT * FROM games');
```

### Persistent connection via `ATTACH`

Best when the remote database should behave like a local DuckDB catalog — supports catalog lookups, `SELECT`, `INSERT`, `COPY`, and `CREATE TABLE AS` (CTAS).

```sql
ATTACH 'profile://mydb' AS mydb (TYPE adbc);
USE mydb.main;

SHOW ALL TABLES;
SELECT * FROM games;
INSERT INTO games (SELECT 6, 'Battleship', 'Clifford Von Wickler', 1931, 7, 2, 2, 12.99);

-- Move data into or out of local DuckDB tables
CREATE TABLE memory.inventors AS (SELECT id, inventor FROM games);
CREATE TABLE game_inventors(id, inventor) AS (SELECT * FROM memory.inventors);
```

For systems with non-default schema/table delimiters (e.g. SQL Server's `[schema].[table]`):

```sql
ATTACH 'profile://mydb' AS mydb (TYPE adbc, DELIMITER '[]');
```

### Arbitrary DDL/DML via `adbc_execute`

For statements that don't fit the `SELECT`/`INSERT`/`COPY`/CTAS surface (e.g. `DROP TABLE`, `CREATE INDEX`):

```sql
CALL adbc_execute('profile://mydb', 'DROP TABLE games');
```

### Refreshing cached metadata

DuckDB caches schema/table metadata from attached ADBC databases locally. After a remote schema change, clear it:

```sql
CALL adbc_clear_cache();
```

### Mixing ADBC reads and writes in one statement

Disabled by default to avoid concurrency bugs. To allow it:

```sql
SET adbc_mix_reads_writes = true;
```

To materialize input rows before `INSERT`/CTAS (also helps avoid concurrency issues when mixing reads/writes):

```sql
SET adbc_materialize_insert_rows = true;
```

### Tuning connection pooling and insert batching

```sql
-- Max pooled connections per attached database (default 50)
SET adbc_connection_pool_size = 100;

-- Chunks buffered in memory before an ADBC bulk insert (default 1000, i.e. ~2M rows @ 2048 rows/chunk)
SET adbc_insert_buffer_size = 10000;
```

## Limitations

- **Autocommit only.** Every statement takes effect immediately; there is no manual transaction control.
- **No automatic pushdown.** Predicate/projection pushdown isn't performed for attached (`ATTACH`) tables — use `read_adbc` with a scoped SQL query when pushdown matters.
- **No DuckDB-to-DuckDB.** The extension doesn't currently support connecting to another DuckDB (or Quack) instance via ADBC.
- **No intra-process concurrency.** Concurrent ADBC operations within a single process aren't supported.
- **Mixed reads/writes need an opt-in.** See `adbc_mix_reads_writes` above.

## Troubleshooting

- **Extension won't load / `INSTALL adbc FROM community` fails.** Confirm the DuckDB version is 1.4.5+ or 1.5.4+.
- **Driver not found / failed to load.** Install it with `dbc install <system>`; verify the `driver` value in the profile matches the name `dbc` uses.
- **Profile not found (`profile://name` fails).** Confirm the `.toml` file is saved in the correct per-OS ADBC profile directory, or reference it by absolute path instead of name.
- **Authentication errors.** Double-check the `uri` and any credential fields under `[Options]` in the profile. For databases needing certificates, tokens, or extra parameters, consult the driver's own documentation via `dbc`.
- **Unexpected "Not implemented" error on INSERT.** Likely a mixed read/write statement — set `adbc_mix_reads_writes = true` (and consider `adbc_materialize_insert_rows = true`).
- **Stale results after an external schema change.** Run `CALL adbc_clear_cache();`.