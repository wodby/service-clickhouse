# ClickHouse on Wodby

What Wodby sets up for this ClickHouse service. It runs the official `clickhouse/clickhouse-server` image as a single node.

## How applications reach it

- Host: the name of this app service inside the environment.
- Ports: `8123` HTTP interface, `9000` native protocol. Applications inside the environment use these directly, without TLS.
- Wodby defines three tokens: `database` (`default`), `username` (a fixed account name) and `password` (generated). A service linked to this one receives host, port and these tokens in variables named by the linking service; read them in code and do not copy the password into the repository.
- Inside this container the same values are in `CLICKHOUSE_DB`, `CLICKHOUSE_USER` and `CLICKHOUSE_PASSWORD`. The image creates that user and database from them.
- `CLICKHOUSE_DEFAULT_ACCESS_MANAGEMENT` is set to `1`, so this user may create other users, roles and databases with SQL.

## Public address

Port `8123` is the service's main HTTP endpoint, so it gets a public HTTPS address. `9000` stays private.

- The address's root answers `Ok.`. Open `/play` to run queries and `/dashboard` for the built-in charts, and sign in there with the account above.
- The public address accepts SQL from anywhere for anyone with that password. Queries sent without it are refused.
- Do not turn on Wodby Basic Auth for this service. ClickHouse uses the `Authorization` header for its own login, so `/play`, `/dashboard` and HTTP clients stop working behind Basic Auth. To restrict who can reach the address, use App Access or disable the route.
- `/play` and `/dashboard` send the password in the request URL, so it can appear in ingress logs.

## Changing configuration

Server settings are not exposed as manifest settings or config files. Create further databases, users and grants with SQL through the account above rather than through additional environment variables.

## Data

Everything is in `/var/lib/clickhouse` on the `data` volume. The manifest declares no backups, imports or actions.

## Check the result

- From any service in the environment: `GET http://<app service name>:8123/ping` answers `Ok.`
- From this service's container: `clickhouse-client --user "$CLICKHOUSE_USER" --password "$CLICKHOUSE_PASSWORD" --query "SHOW DATABASES"`
