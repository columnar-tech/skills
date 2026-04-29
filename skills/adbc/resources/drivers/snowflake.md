# snowflake

## Installing the Driver

```sh
dbc install snowflake
```

## Before Connecting: Determine the Auth Method and Parameters

Snowflake supports several authentication methods, each with different required parameters. Before writing code:

1. **Check for a connection profile.** If the user has one (see `resources/connection-profiles.md`), prefer `profile://<name>` and skip the rest.
2. **Check the environment** for common Snowflake variables (listed below), including a `.env` file if present. If enough are set to satisfy one auth method unambiguously, read them at runtime (e.g., `os.environ`) — do not hardcode values.
3. **Ask the user** for any missing required parameters, and ask which auth method to use if still ambiguous. Never invent or use placeholder values for accounts, usernames, passwords, keys, or tokens.

### Common Environment Variables (Snowflake convention; not read automatically by the driver)

- `SNOWFLAKE_ACCOUNT` — account identifier
- `SNOWFLAKE_USER` — username
- `SNOWFLAKE_PASSWORD` — password
- `SNOWFLAKE_AUTHENTICATOR` — auth method override
- `SNOWFLAKE_PRIVATE_KEY_PATH` — PEM-encoded RSA private key path (JWT auth)
- `SNOWFLAKE_PAT` or `SNOWFLAKE_AUTH_TOKEN` — programmatic access token (PAT auth)
- `SNOWFLAKE_WAREHOUSE`, `SNOWFLAKE_DATABASE`, `SNOWFLAKE_SCHEMA`, `SNOWFLAKE_ROLE`

## Connection Styles

Pick **one** style and use it consistently.

### URI style

Set the `uri` option:

```text
snowflake://[user[:password]@]<account_identifier>/[database][?param1=value1&...]
```

URI query parameters: `warehouse`, `role`, `authenticator`, `token` (for PAT), plus the `adbc.snowflake.sql.*` options below as `&key=value` pairs. The auth method goes in `?authenticator=<value>` (see table below).

### Options style

Omit `uri` and pass these keys directly:

| Key | Purpose |
| --- | --- |
| `adbc.snowflake.sql.account` | Account identifier |
| `username` | Username |
| `password` | Password (secret) |
| `adbc.snowflake.sql.auth_type` | Auth method (see table below) |
| `adbc.snowflake.sql.db` | Default database |
| `adbc.snowflake.sql.schema` | Default schema |
| `adbc.snowflake.sql.warehouse` | Default warehouse |
| `adbc.snowflake.sql.role` | Default role |
| `adbc.snowflake.sql.client_option.tls_skip_verify` | `"true"` to disable TLS verification (not for production) |

### Auth method value mapping

The two styles use different value names for the same auth methods:

| Auth method | Options: `adbc.snowflake.sql.auth_type` | URI: `authenticator` |
| --- | --- | --- |
| Username / password (default) | `auth_snowflake` | `snowflake` |
| JWT key pair | `auth_jwt` | `snowflake_jwt` |
| Programmatic access token (PAT) | `auth_pat` | `programmatic_access_token` |
| OAuth | `auth_oauth` | `oauth` |
| External browser (SSO) | `auth_ext_browser` | `externalbrowser` |
| Okta | `auth_okta` | `okta_endpoint` |
| Username / password + MFA | `auth_mfa` | `username_password_mfa` |
| Workload Identity Federation | `auth_wif` | *(consult upstream docs)* |

Auth info must match the chosen style: when `uri` is set, `authenticator` goes in the URI query string; when using options, set `adbc.snowflake.sql.auth_type`.

## Auth Methods and Their Required Parameters

### 1. Username / Password (default)

- `adbc.snowflake.sql.account`
- `username`
- `password`

`auth_snowflake` is the default, so the auth marker is optional. URI equivalent: `snowflake://<user>:<password>@<account>/[database]`.

### 2. JWT Key Pair (RSA private key)

- `adbc.snowflake.sql.account`
- `username`
- Auth marker: `auth_type=auth_jwt` (options) or `authenticator=snowflake_jwt` (URI)
- Exactly one private key source:
  - `adbc.snowflake.sql.client_option.jwt_private_key` — path to a PEM-encoded RSA private key file, **or**
  - `adbc.snowflake.sql.client_option.jwt_private_key_pkcs8_value` — the PEM contents inline (secret)

### 3. Programmatic Access Token (PAT)

- `adbc.snowflake.sql.account`
- `username`
- Auth marker: `auth_type=auth_pat` (options) or `authenticator=programmatic_access_token` (URI)
- Token delivery:
  - Options style: `adbc.snowflake.sql.client_option.auth_token` = `<PAT>`
  - URI style: `&token=<url-encoded PAT>` in the query string

### 4. Other Authenticators

`auth_oauth`, `auth_ext_browser` (SSO via browser), `auth_okta`, `auth_mfa`, and `auth_wif` (Workload Identity Federation) have their own required parameters — consult https://docs.adbc-drivers.org/drivers/snowflake/index.html and ask the user for the specific values (e.g., OAuth token, Okta URL) rather than guessing option keys.

## Selecting a Database

If no database was specified via URI or `adbc.snowflake.sql.db`, list available databases with `AdbcConnectionGetObjects` at `depth="catalogs"`, then execute `USE DATABASE <NAME>`. Schema, warehouse, and role can also be switched with `USE SCHEMA`, `USE WAREHOUSE`, and `USE ROLE`.

## More Information

See https://docs.adbc-drivers.org/drivers/snowflake/index.html for more detailed documentation if needed.
