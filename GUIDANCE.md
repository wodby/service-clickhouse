# ClickHouse on Wodby

What Wodby sets up for this ClickHouse service. It runs the official `clickhouse/clickhouse-server` image as a single node.

## How applications reach it

- Host: the name of this app service inside the environment.
- Ports: `8123` HTTP interface, `9000` native protocol. Both are private: no public route is created. Neither uses TLS.
- Wodby defines three tokens: `database` (`default`), `username` (a fixed account name) and `password` (generated). A service linked to this one receives host, port and these tokens in variables named by the linking service; read them in code and do not copy the password into the repository.
- Inside this container the same values are in `CLICKHOUSE_DB`, `CLICKHOUSE_USER` and `CLICKHOUSE_PASSWORD`. The image creates that user and database from them.
- `CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT` is set to `1`, so this user may create other users, roles and databases with SQL.

## Changing configuration

Server settings are not exposed as manifest settings or config files. Create further databases, users and grants with SQL through the account above rather than through additional environment variables.

## Data

Everything is in `/var/lib/clickhouse` on the `data` volume. The manifest declares no backups, imports or actions.

## Check the result

- From any service in the environment: `GET http://<app service name>:8123/ping` answers `Ok.`
- From this service's container: `clickhouse-client --user "$CLICKHOUSE_USER" --password "$CLICKHOUSE_PASSWORD" --query "SHOW DATABASES"`
