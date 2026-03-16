# snowflake

## Installing the Driver

Install the Snowflake ADBC driver with dbc by running:

```sh
dbc install snowflake
```

## Connecting

Connect with the following URI syntax:

```text
snowflake://username[:password]@<account_identifier>/dbname/schemaname[?param1=value&...&paramN=valueN]
```

See https://docs.adbc-drivers.org/drivers/snowflake/index.html for more options.

## Selecting a database

If `dbname` was not provided in the connection URI earlier, a database must be selected by executing the following commands.

List available databases using `AdbcConnectionGetObjects` with `depth` set to "catalog".

Then select a database by executing:

```sql
USE DATABASE <NAME>
```

## More Information

See https://docs.adbc-drivers.org/drivers/snowflake/index.html for more detailed documentation if needed.
