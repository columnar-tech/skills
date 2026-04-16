# r

## Suggested Packages

- adbcdrivermanager
- arrow
- tibble
- dplyr (optional, for data manipulation)

## Usage

```r
library(adbcdrivermanager)

# Load driver
drv <- adbc_driver("sqlite")

# Connect to database
db <- adbc_database_init(drv, uri = ":memory:")
con <- adbc_connection_init(db)

# Execute a query
con |> read_adbc("SELECT 41") |> tibble::as_tibble() |> print()

# Execute a query with a bind parameter
con |>
  read_adbc("SELECT ? + 1 AS the_answer", bind = data.frame(x = 41)) |>
  tibble::as_tibble() |>
  print()

# Ingest a Parquet file and read it back
library(arrow)
arrow::read_parquet("../penguins.parquet") |> write_adbc(con, "penguins")
con |> read_adbc("SELECT COUNT(*) AS total_rows FROM penguins") |> tibble::as_tibble() |> print()

# List all catalogs
adbc_connection_get_objects(con, depth = 1L) |> as.data.frame() |> print()

# List all schemas in a specific catalog
adbc_connection_get_objects(con, depth = 2L, catalog = "main") |> as.data.frame() |> print()

# List all tables in a specific schema
adbc_connection_get_objects(con, depth = 3L, catalog = "main", db_schema = "") |> as.data.frame() |> print()

# Clean up
adbc_connection_release(con)
adbc_database_release(db)
```

## Connection Profiles

The R `adbcdrivermanager` package compiles the C++ driver manager with profile support included. The exact R-level API to pass a `profile://` URI has not been verified with a dedicated R test in the upstream repo. Based on the C++ driver manager behavior the expected pattern is:

```r
library(adbcdrivermanager)

db <- adbc_database_init(
  adbc_driver("adbc_driver_manager"),
  uri = "profile://mydb_dev"
)
con <- adbc_connection_init(db)
```

Set `ADBC_PROFILE_PATH` in the environment to point to the directory containing your `.toml` profile files. See `resources/connection-profiles.md` for TOML format and file locations.

## More information

See https://arrow.apache.org/adbc/current/r/index.html for more detailed documentation if needed.
