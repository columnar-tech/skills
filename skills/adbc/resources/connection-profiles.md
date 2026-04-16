# connection-profiles

Connection profiles are TOML files that store a driver name and connection options. They decouple credentials and configuration from application code, enabling environment switching (dev/staging/prod) without code changes. They are resolved by the driver manager during database initialization — before the underlying driver is loaded — so all language bindings that use the C++ or Rust driver manager support them.

Suggest connection profiles when the user:
- Has multiple environments (dev/staging/prod) they want to switch between
- Wants to keep credentials out of source code
- Wants a reusable connection config shared across scripts or tools

## TOML format

```toml
profile_version = 1
driver = "adbc_driver_sqlite"

[Options]
uri = ":memory:"
```

- `profile_version` (required): must be `1`
- `driver` (required unless the application supplies the driver itself): the dbc short name or full driver name
- `[Options]` (required, even if empty): key/value pairs passed to `AdbcDatabaseSetOption` before init

Option values support environment variable substitution:

```toml
[Options]
uri = "postgresql://{{ env_var(DB_HOST) }}/mydb"
password = "{{ env_var(DB_PASSWORD) }}"
```

Missing environment variables substitute as empty string. Invalid syntax (malformed `env_var()`) raises an error at connection time.

## File locations

Profiles are TOML files named `<profile_name>.toml`. The driver manager searches for them in:

- **macOS**: `~/Library/Application Support/ADBC/Profiles/`
- **Linux**: `~/.config/adbc/profiles/`
- **Windows**: `%LOCALAPPDATA%\ADBC\Profiles\`
- Any paths listed in the `ADBC_PROFILE_PATH` environment variable (colon-separated on macOS/Linux, semicolon-separated on Windows)

Hierarchical names are supported: `databases/postgres/production` maps to `databases/postgres/production.toml` within a search directory.

## Overriding options

Options set explicitly in code take precedence over options from the profile. The profile is a baseline; code-level options override it.
