# python

## Notes

Prefer to use `uv` if the user has it installed. When running scripts, prefer to use PEP 723 style comments at the top to define dependencies:

```python
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.10"
# dependencies = [
#     "dependency-name",
# ]
# ///
```

and then use `uv run script.py` to run the script.

## Suggested Packages

- adbc-driver-manager
- dotenv
- pandas
- pyarrow

## Usage

```python
#!/usr/bin/env -S uv run
# /// script
# requires-python = ">=3.10"
# dependencies = ["adbc-driver-manager>=1.9.0", "pyarrow>=20.0.0"]
# ///

from adbc_driver_manager import dbapi
import pyarrow.parquet as pq

# Load driver and connect to database
with dbapi.connect(driver="sqlite", db_kwargs={"uri": ":memory:"}) as con:
    with con.cursor() as cursor:
        # Execute a query
        cursor.execute("SELECT 41")
        print(cursor.fetch_arrow_table())

        # Execute a query with a bind parameter
        cursor.execute("SELECT ? + 1 AS the_answer", parameters=(41,))
        print(cursor.fetch_arrow_table())

        # Ingest a Parquet file and read it back
        penguins = pq.read_table("../penguins.parquet")
        cursor.adbc_ingest("penguins", penguins)

        # Important! dbapi does not autocommit by default so we should commit
        con.commit()

        cursor.execute("SELECT COUNT(*) AS total_rows FROM penguins")
        print(cursor.fetch_arrow_table())

        # List all catalogs
        print(con.adbc_get_objects(depth="catalogs").read_all())

        # List all schemas in a specific catalog
        print(con.adbc_get_objects(depth="db_schemas", catalog_filter="main").read_all())

        # List all tables in a specific schema
        print(con.adbc_get_objects(depth="tables", catalog_filter="main", db_schema_filter="").read_all())
```

## More information

See https://arrow.apache.org/adbc/current/python/index.html for more detailed documentation if needed.
