# snowflake

## Installing the Driver

Install the Snowflake ADBC driver with dbc by running:

```sh
dbc install snowflake
```

## Before Connecting: Determine the Auth Method and Parameters

Snowflake supports several authentication methods, each of which requires a different set of connection parameters. Before writing code, determine which auth method to use and gather the required parameters. Follow this order:

1. **Check for a connection profile.** If the user has a profile (see `resources/connection-profiles.md`), prefer `profile://<name>` and skip the rest.
2. **Check the environment for common Snowflake variables** (listed below). If enough are present to satisfy one auth method unambiguously, use them via `os.environ` (Python) or the equivalent in other languages. Do not hardcode their values into the script.
3. **Ask the user** for any missing required parameters, and ask which auth method they want if it is still ambiguous. Never invent, guess, or use placeholder values for account identifiers, usernames, passwords, or key paths.

### Common Environment Variables to Check

These are the conventional names used by Snowflake's own tools (SnowSQL, the Python connector, etc.). They are not read by the ADBC driver automatically — you must pass them into the connection options — but they are a good first place to look:

- `SNOWFLAKE_ACCOUNT` — account identifier
- `SNOWFLAKE_USER` — username
- `SNOWFLAKE_PASSWORD` — password
- `SNOWFLAKE_AUTHENTICATOR` — auth method override (see values below)
- `SNOWFLAKE_PRIVATE_KEY_PATH` — path to PEM-encoded RSA private key (for JWT auth)
- `SNOWFLAKE_WAREHOUSE` — default warehouse
- `SNOWFLAKE_DATABASE` — default database
- `SNOWFLAKE_SCHEMA` — default schema
- `SNOWFLAKE_ROLE` — default role

If a `.env` file is present in the project, read it (e.g., with `python-dotenv`) before checking `os.environ`.

## Connecting by URI

The Snowflake URI format is:

```text
snowflake://[user[:password]@]<account_identifier>/[database][?param1=value1&...]
```

URI components:

- `user` (optional in the URI; can instead be set via the `username` option)
- `password` (optional; secret — prefer env var over inlining)
- `<account_identifier>` (**required**, e.g. `myorg-account123`)
- `database` (optional)

Supported URI query parameters include:

- `warehouse` — virtual warehouse for queries (e.g., `COMPUTE_WH`)
- `authenticator` — auth method override, one of: `snowflake`, `externalbrowser`, `okta_endpoint`, `oauth`, `username_password_mfa`, `snowflake_jwt`

## Connecting by Options

Instead of (or in addition to) a URI, pass these options to the driver. Option keys used across all auth methods:

| Key | Purpose |
| --- | --- |
| `adbc.snowflake.sql.account` | Account identifier |
| `username` | Username |
| `password` | Password (secret) |
| `adbc.snowflake.sql.db` | Default database |
| `adbc.snowflake.sql.schema` | Default schema |
| `adbc.snowflake.sql.warehouse` | Default warehouse |
| `adbc.snowflake.sql.role` | Default role |
| `adbc.snowflake.sql.client_option.tls_skip_verify` | `"true"` to disable TLS verification (not for production) |

## Auth Methods and Their Required Parameters

### 1. Username / Password (default)

Standard user credentials. Required:

- `adbc.snowflake.sql.account` — account identifier
- `username`
- `password`

Or equivalently via URI: `snowflake://<user>:<password>@<account>/[database]`

### 2. JWT Key Pair (RSA private key)

Authenticate with an RSA private key. Required:

- `adbc.snowflake.sql.account` — account identifier
- `username`
- Exactly one of the following (the "private key source"):
  - `adbc.snowflake.sql.client_option.jwt_private_key` — path to a PEM-encoded RSA private key file, **or**
  - `adbc.snowflake.sql.client_option.jwt_private_key_pkcs8_value` — the PEM-encoded private key contents pasted directly (secret)

Set `authenticator=snowflake_jwt` (via URI query or the equivalent driver option) to force JWT auth.

### 3. Other Authenticators

The driver also accepts these values for the `authenticator` URI parameter: `externalbrowser` (SSO via browser), `okta_endpoint` (native Okta), `oauth`, and `username_password_mfa`. Each of these has its own required parameters — consult https://docs.adbc-drivers.org/drivers/snowflake/index.html before using them, and ask the user for the specific values they need (e.g., OAuth token, Okta URL) rather than guessing option keys.

## Selecting a Database

If no database was specified in the URI or via `adbc.snowflake.sql.db`, list available databases with `AdbcConnectionGetObjects` at `depth="catalogs"`, then select one by executing:

```sql
USE DATABASE <NAME>
```

The schema, warehouse, and role can likewise be switched with `USE SCHEMA`, `USE WAREHOUSE`, and `USE ROLE` after connecting.

## More Information

See https://docs.adbc-drivers.org/drivers/snowflake/index.html for more detailed documentation if needed.
