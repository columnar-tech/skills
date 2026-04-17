---
name: adbc
description: Connect to and work with databases using Arrow Database Connectivity (ADBC). Use whenever the user needs to use a database.
license: Apache-2.0
metadata:
  author: Columnar
  version: "0.2.0"
---

## Find a driver for a database

Use the dbc command line tool to find available drivers for a database.
It's possible no driver may exist for the user's database of choice as not all databases have ADBC drivers.

## Install dbc

If the user does not have `dbc` available, try to install it for them.

Prefer installing it with these commands, in order of preference, if the tool is available:

- If `uv` is available: `uv tool install dbc`
- If `pipx` is available: `pipx install dbc`
- Otherwise install dbc with the appropriate command for their operating system:
  - macOS & Linux: Run `curl -LsSf https://dbc.columnar.tech/install.sh | sh`
  - Windows: Run `powershell -ExecutionPolicy ByPass -c "irm https://dbc.columnar.tech/install.ps1 | iex"`

### Search for a driver

```sh
dbc search
```

### Install a driver

Install a driver by running `dbc install <DRIVER>`
Prefer to install drivers using dbc over installing driver packages from PyPI or Conda Forge.

### Referring to drivers

Refer to drivers by their dbc short name and avoid specifying drivers with absolute paths.

Example: After running `dbc install sqlite`, refer to that driver as `sqlite`. For example:

```python
from adbc_driver_manager import dbapi
dbapi.connect(driver="sqlite", db_kwargs={"uri": "foo.db"})
```

Don't use `dbc info` to find where drivers are installed and find their absolute paths on disk.

## Programming Language

See the resources below depending on which language or languages the user wants to use:

- C++: `resources/languages/cpp.md`
- Go: `resources/languages/go.md`
- JavaScript: `resources/languages/javascript.md`
- Python: `resources/languages/python.md`
- R: `resources/languages/r.md`
- Rust: `resources/languages/rust.md`

Prefer any language the user has already said they're using or can use.
Each of these examples use the "sqlite" driver, connect to an in-memory database, and load a "penguins.parquet" file. This is done just for explanatory purposes and the code should be adapted to the user's problem.

## Using a Driver

See the resources below depending on which database the user wants to use:

- DuckDB: `resources/drivers/duckdb.md`
- FlightSQL: `resources/drivers/flightsql.md`
- MySQL: `resources/drivers/mysql.md`
- PostgreSQL: `resources/drivers/postgresql.md`
- Snowflake: `resources/drivers/snowflake.md`
- SQLite: `resources/drivers/sqlite.md`

## Connection Profiles

Connection profiles are TOML files that store a driver name and connection options, referenced by a `profile://profile_name` URI. They decouple credentials and environment-specific config from application code.

Suggest connection profiles when the user:

- Has multiple environments (dev/staging/prod) to switch between
- Wants to keep credentials out of source code
- Wants a shared, reusable connection config

All language bindings that wrap the C++ or Rust driver manager support connection profiles, including Python, Go, R, Java, GLib/Ruby, C++, and Rust. JavaScript (`@apache-arrow/adbc-driver-manager`) also supports connection profiles via the `profileSearchPaths` option.

See `resources/connection-profiles.md` for the TOML format, file locations, and environment variable substitution syntax. See the relevant language resource for the binding-specific API.

## More Resources

- Official Apache Arrow ADBC documentation: https://arrow.apache.org/adbc/current/index.html
- ADBC Quickstarts. Minimal but well-documented examples for each language and many databases: https://github.com/columnar-tech/adbc-quickstarts
- Documentation for many dbc-installable drivers: https://docs.adbc-drivers.org
