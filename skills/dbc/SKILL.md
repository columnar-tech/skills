---
name: dbc
description: Install ADBC (Arrow Database Connectivity) drivers with dbc. Use when the user wants to install database drivers and connect to databases.
license: Apache-2.0
metadata:
  author: Columnar
  version: "1.1.1"
---

# dbc Skill

Install and manage drivers for the user with the dbc command line program.

## Installing dbc

If the user does not have `dbc` available, try to install it for them.

Prefer installing it with these commands, in order of preference, if the tool is available:

- If `uv` is available: `uv tool install dbc`
- If `pipx` is available: `pipx install dbc`
- If `brew` is available: `brew install columnar-tech/tap/dbc`
- On Windows, if `winget` is available: `winget install dbc`
- Otherwise, direct the user to the installation docs: https://docs.columnar.tech/dbc/getting_started/installation/

## Most Important Commands

- `dbc install <driver>` - Install a driver (e.g., `dbc install snowflake`)
- `dbc search [pattern]` - Search for a driver using a pattern (e.g., `dbc search sql`). Also lists installed drivers (see below).
- `dbc info <driver>` - Get driver details

Run `dbc --help` to learn about other commands.

## Listing Installed Drivers

**There is no `dbc list` command. Do not run `dbc list` — it will fail.**

To see which drivers are already installed, run `dbc search` with no arguments (or with a pattern) and look for entries marked `[installed: ...]` in the output. For example:

```
postgresql   An ADBC driver for PostgreSQL ...   [installed: user=>1.11.0]
snowflake    An ADBC driver for Snowflake  ...   [installed: user=>1.10.3]
```

Entries without an `[installed: ...]` tag are available but not yet installed.

## Project Workflow

For reproducible driver management, prefer using this workflow over `dbc install`:

1. `dbc init` - Create a `dbc.toml` file
2. `dbc add <driver>` - Add drivers to the list (supports version constraints like `dbc add "postgresql>=13.0"`)
3. `dbc sync` - Install all drivers and create `dbc.lock`

## Using Drivers

Drivers installed with dbc must be used with an ADBC driver manager. dbc cannot load a driver or query a data source directly. Don't install drivers from any other source like PyPI, prefer drivers installed with dbc always.

Resources for using drivers:

- How to load drivers and connect to databases: https://github.com/columnar-tech/adbc-quickstarts
- Python Cookbooks for ADBC: https://arrow.apache.org/adbc/current/python/recipe/index.html
