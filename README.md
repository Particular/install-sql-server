# install-sql-server

This action installs and runs SQL Server for a GitHub Actions workflow.

1. Installs SQL Server
  * On Windows, uses [Chocolatey](https://chocolatey.org/) to install [SQL Server Express](https://community.chocolatey.org/packages/sql-server-express).
  * On Linux, runs SQL as a Docker container using the `mcr.microsoft.com/mssql/server:2022-latest` image.
1. Creates environement variables for a connection string and for `sqlcmd`.
1. Waits for the SQL instance to be accessible.
1. Creates a default database catalog.

## Usage

Install SQL Server 2022 with a default database of `nservicebus` and put the connection string in the environment variable `SQL_SERVER_CONNECTION_STRING`:

```yaml
steps:
  - name: Install SQL Server
    uses: Particular/install-sql-server-action@v1.0.0
    with:
      connection-string-env-var: SQL_SERVER_CONNECTION_STRING
      catalog: nservicebus
```

It is also possible to specify the SQl server major version to be installed

```yaml
steps:
  - name: Install SQL Server
    uses: Particular/install-sql-server-action@v1.2.0
    with:
      connection-string-env-var: SQL_SERVER_CONNECTION_STRING
      sqlserver-version: 2019
      catalog: nservicebus
```

To add additional parameters to the end of the connection string, such as `Max Pool Size`. The connection string generated by the action already ends with a semicolon, and the `extra-params` are appended at the end.

```yaml
steps:
  - name: Install SQL Server
    uses: Particular/install-sql-server-action@v1.0.0
    with:
      connection-string-env-var: SQL_SERVER_CONNECTION_STRING
      catalog: nservicebus
      extra-params: "Max Pool Size=100;"
```

## Parameters

| Parameter | Required | Default | Description |
|-|:-:|:-:|-|
| `connection-string-env-var` | Yes | - | Environment variable name that will be filled with the SQL connection string, which can then be used in successive steps. |
| `catalog` | No | `nservicebus` | The default catalog, which will be created by the action. |
| `sqlserver-version` | No | "2022" | The SQL server major version to use. |
| `extra-params` | No | - | Extra parameters to be appended to the end of the connection string |

## Using `sqlcmd`

The action creates the necessary environment variables so that `sqlcmd` can be used without any additional login parameters.

For example:

```yaml
steps:
  - name: Install SQL Server
    uses: Particular/install-sql-server-action@v1.0.0
    with:
      connection-string-env-var: SQL_SERVER_CONNECTION_STRING
      catalog: nservicebus
  - name: Create schemas
    shell: pwsh
    run: |
        echo "Create additional schemas"
        sqlcmd -Q "CREATE SCHEMA receiver AUTHORIZATION db_owner" -d "nservicebus"
        sqlcmd -Q "CREATE SCHEMA sender AUTHORIZATION db_owner" -d "nservicebus"
        sqlcmd -Q "CREATE SCHEMA db@ AUTHORIZATION db_owner" -d "nservicebus"
```
