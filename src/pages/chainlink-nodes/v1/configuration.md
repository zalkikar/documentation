---
layout: ../../../layouts/MainLayout.astro
section: nodeOperator
date: Last Modified
title: "Configuring Chainlink Nodes"
---

Recent versions of the Chainlink node use sensible defaults for most configuration variables. You do not need to change much to get a standard deployment working.

Not all environment variables are documented here. Any undocumented environment variable is subject to change in future releases. In almost all cases, you should leave any environment variable not listed here to its default value unless you really understand what you are doing.

To reiterate: _If you have an environment variable set that is not listed here, and you don't know exactly why you have it set, you should remove it!_

The environment variables listed here are explicitly supported and current as of Chainlink node v1.3.0.

### TOML Configuration

TOML configuration for Chainlink nodes is stable and recommended for mainnet deployments. TOML configuration will be the only supported configuration method starting with `v2.0.0`. Enable TOML configuration by specifying the `-config <filename>.toml` flag with the path to your TOML file. Alternatively, you can specify the raw TOML config in the [`CL_CONFIG` environment variable](/chainlink-nodes/v1/configuration#cl_config). See the [CONFIG.md](https://github.com/smartcontractkit/chainlink/blob/v1.13.0/docs/CONFIG.md) and [SECRETS.md](https://github.com/smartcontractkit/chainlink/blob/v1.13.0/docs/SECRETS.md) on GitHub to learn more.

## Changes to node configuration starting in v1.1.0 nodes

As of Chainlink node v1.1.0 and up, the way nodes manage configuration is changing. Previously, environment variables exclusively handled all node configuration. Although this configuration method worked well in the past, it has its limitations. Notably, it doesn't mesh well with chain-specific configuration profiles.

For this reason, Chainlink nodes are moving towards a model where you set variables using the API, CLI, or GUI, and the configuration is saved in the database. We encourage you to become familiar with this model because it is likely that nodes will continue to move away from environment variable configuration in the future.

As of v1.1.0, Chainlink nodes still support environment variables to configure node settings and chain-specific settings. If the environment variable is set, it overrides any chain-specific, job-specific, or database configuration setting. The log displays a warning to indicate when an override happens, so you know when variables lower in the hierarchy are being ignored.

Your node applies configuration settings using following hierarchy:

1. Environment variables
1. Chain-specific variables
1. Job-specific variables

## Essential environment variables

These are the only environment variables that are _required_ for a Chainlink node to run.

### DATABASE_URL

**Required**

- Default: _none_

The PostgreSQL URI to connect to your database. Chainlink nodes require Postgres versions >= 11. See the [Running a Chainlink Node](/chainlink-nodes/v1/running-a-chainlink-node) for an example.

## General Node Configuration

### CL_CONFIG

This environment variable is used to set static configuration using TOML format. Specify the raw TOML config in this environment variable. Unlike the `-config` flag, it does not accept a path to a TOML file.

See the [CONFIG.md](https://github.com/smartcontractkit/chainlink/blob/v1.13.0/docs/CONFIG.md) and [SECRETS.md](https://github.com/smartcontractkit/chainlink/blob/v1.13.0/docs/SECRETS.md) on GitHub to learn more.

### CHAIN_TYPE

- Default: _none_

CHAIN_TYPE overrides all chains and forces them to act as a particular chain type. An up-to-date list of chain types is given in [`chaintype.go`](https://github.com/smartcontractkit/chainlink/blob/v1.3.0/core/chains/chaintype.go).

This variable enables some chain-specific hacks and optimizations. It is recommended not to use this environment variable and set the chain-type on a per-chain basis instead.

### CHAINLINK_DEV

- Default: `"false"`

Setting `CHAINLINK_DEV` to `true` enables development mode. Do not use this for production deployments. It can be useful for enabling experimental features and collecting debug information in test environments.

### EXPLORER_ACCESS_KEY

- Default: _none_

The access key for authenticating with the explorer. This variable is required to deliver telemetry.

### EXPLORER_SECRET

- Default: _none_

The secret for authenticating with the explorer. This variable is required to deliver telemetry.

### EXPLORER_URL

- Default: _none_

The explorer websocket URL for the node to push stats to. This variable is required to deliver telemetry.

### ROOT

- Default: `"~/.chainlink"`

The Chainlink node's root directory. This is the default directory for logging, database backups, cookies, and other misc Chainlink node files. Chainlink nodes will always ensure this directory has `700` permissions because it might contain sensitive data.

### TELEMETRY_INGRESS_UNICONN

- Default: `"true"`

Toggles which ws connection style is used.

### TELEMETRY_INGRESS_LOGGING

- Default: `"false"`

Toggles verbose logging of the raw telemetry messages being sent.

### TELEMETRY_INGRESS_URL

- Default: _none_

The URL to connect to for sending telemetry.

### TELEMETRY_INGRESS_SERVER_PUB_KEY

- Default: _none_

The public key of the telemetry server.

### TELEMETRY_INGRESS_BUFFER_SIZE

- Default: `"100"`

The number of telemetry messages to buffer before dropping new ones.

### TELEMETRY_INGRESS_MAX_BATCH_SIZE

- Default: `"50"`

The maximum number of messages to batch into one telemetry request.

### TELEMETRY_INGRESS_SEND_INTERVAL

- Default: `"500ms"`

The interval on which batched telemetry is sent to the ingress server.

### TELEMETRY_INGRESS_SEND_TIMEOUT

- Default: `"10s"`

The max duration to wait for the request to complete when sending batch telemetry.

### TELEMETRY_INGRESS_USE_BATCH_SEND

- Default: `"true"`

Toggles sending telemetry to the ingress server using the batch client.

## Chains

### SOLANA_ENABLED

:::caution
Not intended for use on the Solana mainnet.
:::

- Default: `"false"`

Enables Solana support.

### EVM_ENABLED

- Default: `"true"`

Enables support for EVM-based chains. By default, this variable is set to `true` to provide legacy compatibility and ease the upgrade path from older versions of Chainlink which did not support disabling EVM.

## Database Settings

### MIGRATE_DATABASE

- Default: `"true"`

This variable controls whether a Chainlink node will attempt to automatically migrate the database on boot. If you want more control over your database migration process, set this variable to `false` and manually migrate the database using the CLI `migrate` command instead.

### ORM_MAX_IDLE_CONNS

- Default: `"10"`

This setting configures the maximum number of idle database connections that the Chainlink node will keep open. Think of this as the baseline number of database connections per Chainlink node instance. Increasing this number can help to improve performance under database-heavy workloads.

Postgres has connection limits, so you must use cation when increasing this value. If you are running several instances of a Chainlink node or another application on a single database server, you might run out of Postgres connection slots if you raise this value too high.

### ORM_MAX_OPEN_CONNS

- Default: `"20"`

This setting configures the maximum number of database connections that a Chainlink node will have open at any one time. Think of this as the maximum burst upper bound limit of database connections per Chainlink node instance. Increasing this number can help to improve performance under database-heavy workloads.

Postgres has connection limits, so you must use cation when increasing this value. If you are running several instances of a Chainlink node or another application on a single database server, you might run out of Postgres connection slots if you raise this value too high.

## Database Global Lock

Chainlink nodes use a database lock to ensure that only one Chainlink node instance can be run on the database at a time. If you run multiple instances of a Chainlink node that share a single database at the same time, the node will encounter strange errors and data integrity failures. Do not allow multiple nodes to write data to the database at the same time.

### DATABASE_LOCKING_MODE

- Default: `"dual"`

The `DATABASE_LOCKING_MODE` variable can be set to 'dual', 'advisorylock', 'lease', or 'none'. It controls which mode to use to enforce that only one Chainlink node can use the database. It is recommended to set this to `lease`.

- `dual` - The default: Uses both advisory locks and lease locks for backward and forward compatibility
- `advisorylock` - Advisory lock only
- `lease` - Lease lock only
- _none_ - No locking at all: This option useful for advanced deployment environments when you are sure that only one instance of a Chainlink node will ever be running.

#### Technical details

Ideally, you should use a container orchestration system like [Kubernetes](https://kubernetes.io/) to ensure that only one Chainlink node instance can ever use a specific Postgres database. However, some node operators do not have the technical capacity to do this. Common use cases run multiple Chainlink node instances in failover mode as recommended by our official documentation. The first instance takes a lock on the database and subsequent instances will wait trying to take this lock in case the first instance fails.

By default, Chainlink nodes use the `dual` setting to provide both advisory locks and lease locks for backward and forward compatibility. Using advisory locks alone presents the following problems:

- If your nodes or applications hold locks open for several hours or days, Postgres is unable to complete internal cleanup tasks. The Postgres maintainers explicitly discourage holding locks open for long periods of time.
- Advisory locks can silently disappear when you upgrade Postgres, so a new Chainlink node instance can take over even while the old node is still running.
- Advisory locks do not work well with pooling tools such as [pgbouncer](https://www.pgbouncer.org/).
- If the Chainlink node crashes, an advisory lock can hang around for up to several hours, which might require you to manually remove it so another instance of the Chainlink node will allow itself to boot.

Because of the complications with advisory locks, Chainlink nodes with v1.1.0 and later support a new `lease` locking mode. This mode might become the default in future. The `lease` locking mode works using the following process:

- Node A creates one row in the database with the client ID and updates it once per second.
- Node B spinlocks and checks periodically to see if the client ID is too old. If the client ID is not updated after a period of time, node B assumes that node A failed and takes over. Node B becomes the owner of the row and updates the client ID once per second.
- If node A comes back, it attempts to take out a lease, realizes that the database has been leased to another process, and exits the entire application immediately.

### ADVISORY_LOCK_CHECK_INTERVAL

**ADVANCED**

Do not change this setting unless you know what you are doing.

This setting applies only if `DATABASE_LOCKING_MODE` is set to enable advisory locking.

- Default: `"1s"`

`ADVISORY_LOCK_CHECK_INTERVAL` controls how often the Chainlink node checks to make sure it still holds the advisory lock when advisory locking is enabled. If a node no longer holds the lock, it will try to re-acquire it. If the node cannot re-acquire the lock, the application will exit.

### ADVISORY_LOCK_ID

**ADVANCED**

Do not change this setting unless you know what you are doing.

This setting applies only if `DATABASE_LOCKING_MODE` is set to enable advisory locking.

- Default: `"1027321974924625846"`

`ADVISORY_LOCK_ID` is the application advisory lock ID. This must match all other Chainlink nodes that might access this database. It is unlikely you will ever need to change this from the default.

### LEASE_LOCK_DURATION

**ADVANCED**

Do not change this setting unless you know what you are doing.

This setting applies only if `DATABASE_LOCKING_MODE` is set to enable lease locking.

- Default: `"30s"`

How long the lease lock will last before expiring.

### LEASE_LOCK_REFRESH_INTERVAL

**ADVANCED**

Do not change this setting unless you know what you are doing.

This setting applies only if `DATABASE_LOCKING_MODE` is set to enable lease locking.

- Default: `"1s"`

How often to refresh the lease lock. Also controls how often a standby node will check to see if it can grab the lease.

## Database Automatic Backups

As a best practice, take regular database backups in case of accidental data loss. This best practice is especially important when you upgrade your Chainlink node to a new version. Chainlink nodes support automated database backups to make this process easier.

NOTE: Dumps can cause high load and massive database latencies, which will negatively impact the normal functioning of the Chainlink node. For this reason, it is recommended to set a DATABASE_BACKUP_URL and point it to a read replica if you enable automatic backups.

### DATABASE_BACKUP_FREQUENCY

- Default: `"1h"`

If this variable is set to a positive duration and `DATABASE_BACKUP_MODE` is not _none_, the node will dump the database at this regular interval.

Set to `0` to disable periodic backups.

### DATABASE_BACKUP_MODE

- Default: `"none"`

Set the mode for automatic database backups, which can be one of _none_, `lite`, or `full`. If enabled, the Chainlink node will always dump a backup on every boot before running migrations. Additionally, it will automatically take database backups that overwrite the backup file for the given version at regular intervals if `DATABASE_BACKUP_FREQUENCY` is set to a non-zero interval.

_none_ - Disables backups.
`lite` - Dumps small tables including configuration and keys that are essential for the node to function, which excludes historical data like job runs, transaction history, etc.
`full` - Dumps the entire database.

It will write to a file like `$ROOT/backup/cl_backup_<VERSION>.dump`. There is one backup dump file per version of the Chainlink node. If you upgrade the node, it will keep the backup taken right before the upgrade migration so you can restore to an older version if necessary.

### DATABASE_BACKUP_URL

If specified, the automatic database backup will pull from this URL rather than the main `DATABASE_URL`. It is recommended to set this value to a read replica if you have one to avoid excessive load on the main database.

### DATABASE_BACKUP_DIR

This variable sets the directory to use for saving the backup file. Use this if you want to save the backup file in a directory other than the default ROOT directory.

## Logging

### JSON_CONSOLE

- Default: `"false"`

Set this to true to enable JSON logging. Otherwise, the log is saved in a human-friendly console format.

### LOG_FILE_DIR

- Default: `"$ROOT"`

By default, Chainlink nodes write log data to `$ROOT/log.jsonl`. The log directory can be changed by setting this var. For example, `LOG_FILE_DIR=/my/log/directory`.

### LOG_LEVEL

- Default: `"info"`

The `LOG_LEVEL` environment variable determines both what is printed on the screen and what is written to the log file.

The available options are:

- `"debug"`: Useful for forensic debugging of issues.
- `"info"`: High-level informational messages.
- `"warn"`: A mild error occurred that might require non-urgent action. Check these warnings semi-regularly to see if any of them require attention. These warnings usually happen due to factors outside of the control of the node operator. Examples: Unexpected responses from a remote API or misleading networking errors.
- `"error"`: An unexpected error occurred during the regular operation of a well-maintained node. Node operators might need to take action to remedy this error. Check these regularly to see if any of them require attention. Examples: Use of deprecated configuration options or incorrectly configured settings that cause a job to fail.
- `"crit"`: A critical error occurred. The node might be unable to function. Node operators should take immediate action to fix these errors. Examples: The node could not boot because a network socket could not be opened or the database became inaccessible.
- `"panic"`: An exceptional error occurred that could not be handled. If the node is unresponsive, node operators should try to restart their nodes and notify the Chainlink team of a potential bug.
- `"fatal"`: The node encountered an unrecoverable problem and had to exit.

### LOG_SQL

- Default: `"false"`

This setting tells the Chainlink node to log SQL statements made using the default logger. SQL statements will be logged at `debug` level. Not all statements can be logged. The best way to get a true log of all SQL statements is to enable SQL statement logging on Postgres.

### LOG_FILE_MAX_SIZE

- Default: `"5120mb"`

Determines the log file's max size in megabytes before file rotation. Having this not set will disable logging to disk. If your disk doesn't have enough disk space, the logging will pause and the application will log errors until space is available again.

Values must have suffixes with a unit like: `5120mb` (5,120 megabytes). If no unit suffix is provided, the value defaults to `b` (bytes). The list of valid unit suffixes are:

- b (bytes)
- kb (kilobytes)
- mb (megabytes)
- gb (gigabytes)
- tb (terabytes)

### LOG_FILE_MAX_AGE

- Default: `"0"`

Determines the log file's max age in days before file rotation. Keeping this config with the default value will not remove log files based on age.

### LOG_FILE_MAX_BACKUPS

- Default: `"1"`

Determines the maximum number of old log files to retain. Keeping this config with the default value retains all old log files. The `LOG_FILE_MAX_AGE` variable can still cause them to get deleted.

### LOG_UNIX_TS

- Default: _none_

Previous versions of Chainlink nodes wrote JSON logs with a unix timestamp. As of v1.1.0 and up, the default has changed to use ISO8601 timestamps for better readability. Setting `LOG_UNIX_TS=true` will enable the old behavior.

### AUDIT_LOGGER_FORWARD_TO_URL

- Default: _none_

When set, this environment variable configures and enables an optional HTTP logger which is used specifically to send audit log events. Audit logs events are emitted when specific actions are performed by any of the users through the node's API. The value of this variable should be a full URL. Log items will be sent via POST HTTP requests.

There are audit log implemented for the following events:

- Auth & Sessions (new session, login success, login failed, 2FA enrolled, 2FA failed, password reset, password reset failed, etc.)
- CRUD actions for all resources (add/create/delete resources such as bridges, nodes, keys)
- Sensitive actions (keys exported/imported, config changed, log level changed, environment dumped)

A full list of audit log enum types can be found in the source within the `audit` package ([`audit_types.go`](https://github.com/smartcontractkit/chainlink/blob/develop/core/logger/audit/audit_types.go)).

Log events follow this schema:

```
{
    "eventID":  EVENT_ID_ENUM,
    "hostname": HOSTNAME,
    "localIP" : CL_NODE_IP,
    "env" : ENVIRONMENT_NAME,
    "data": ...
}
```

The `AUDIT_LOGGER_*` environment variables configure this optional audit log HTTP forwarder.

### AUDIT_LOGGER_HEADERS

- Default: _none_

An optional list of HTTP headers to be added for every optional audit log event. If the above `AUDIT_LOGGER_FORWARD_TO_URL` is set, audit log events will be POSTed to that URL, and will include headers specified in this environment variable. One example use case is auth for example:
`AUDIT_LOGGER_HEADERS="Authorization||{token}"`

Header keys and values are delimited on `||`, and multiple headers can be added with a forward slash delimiter (`\`). An example of multiple key value pairs:
`AUDIT_LOGGER_HEADERS="Authorization||{token}\Some-Other-Header||{token2}"`

### AUDIT_LOGGER_JSON_WRAPPER_KEY

- Default: _none_

When the audit log HTTP forwarder is enabled, if there is a value set for this optional environment variable then the POST body will be wrapped in a dictionary in a field specified by the value of set variable. This is to help enable specific logging service integrations that may require the event JSON in a special shape. For example: `AUDIT_LOGGER_JSON_WRAPPER_KEY=event` will create the POST body:

```
{
  "event": {
    "eventID":  EVENT_ID_ENUM,
    "hostname": HOSTNAME,
    "localIP" : CL_NODE_IP,
    "env" : ENVIRONMENT_NAME,
    "data": ...
  }
}
```

## Nurse service (auto-pprof)

The Chainlink node is equipped with an internal "nurse" service that can perform automatic `pprof` profiling when the certain resource thresholds are exceeded, such as memory and goroutine count. These profiles are saved to disk to facilitate fine-grained debugging of performance-related issues. In general, if you notice that your node has begun to accumulate profiles, forward them to the Chainlink team.

To learn more about these profiles, read the [Profiling Go programs with pprof](https://jvns.ca/blog/2017/09/24/profiling-go-with-pprof/) guide.

### AUTO_PPROF_ENABLED

- Default: `"false"`

Set to `true` to enable the automatic profiling service.

### AUTO_PPROF_PROFILE_ROOT

Defaults to `$CHAINLINK_ROOT`

The location on disk where pprof profiles will be stored.

### AUTO_PPROF_POLL_INTERVAL

- Default: `"10s"`

The interval at which the node's resources are checked.

### AUTO_PPROF_GATHER_DURATION

- Default: `"10s"`

The duration for which profiles are gathered when profiling starts.

### AUTO_PPROF_GATHER_TRACE_DURATION

- Default: `"5s"`

The duration for which traces are gathered when profiling is kicked off. This is separately configurable because traces are significantly larger than other types of profiles.

### AUTO_PPROF_MAX_PROFILE_SIZE

- Default: `"100mb"`

The maximum amount of disk space that profiles may consume before profiling is disabled.

### AUTO_PPROF_CPU_PROFILE_RATE

- Default: `"1"`

See https://pkg.go.dev/runtime#SetCPUProfileRate.

### AUTO_PPROF_MEM_PROFILE_RATE

- Default: `"1"`

See https://pkg.go.dev/runtime#pkg-variables.

### AUTO_PPROF_BLOCK_PROFILE_RATE

- Default: `"1"`

See https://pkg.go.dev/runtime#SetBlockProfileRate.

### AUTO_PPROF_MUTEX_PROFILE_FRACTION

- Default: `"1"`

See https://pkg.go.dev/runtime#SetMutexProfileFraction.

- Default: `"1"`

### AUTO_PPROF_MEM_THRESHOLD

- Default: `"4gb"`

The maximum amount of memory the node can actively consume before profiling begins.

### AUTO_PPROF_GOROUTINE_THRESHOLD

- Default: `"5000"`

The maximum number of actively-running goroutines the node can spawn before profiling begins.

## Chainlink Web Server

### ALLOW_ORIGINS

- Default: `"http://localhost:3000,http://localhost:6688"`

Controls the URLs Chainlink nodes emit in the `Allow-Origins` header of its API responses. The setting can be a comma-separated list with no spaces. You might experience CORS issues if this is not set correctly.

You should set this to the external URL that you use to access the Chainlink UI.

You can set `ALLOW_ORIGINS=*` to allow the UI to work from any URL, but it is recommended for security reasons to make it explicit instead.

### AUTHENTICATED_RATE_LIMIT

- Default: `"1000"`

`AUTHENTICATED_RATE_LIMIT` defines the threshold to which authenticated requests get limited. More than this many authenticated requests per `AUTHENTICATED_RATE_LIMIT_PERIOD` will be rejected.

### AUTHENTICATED_RATE_LIMIT_PERIOD

- Default: `"1m"`

`AUTHENTICATED_RATE_LIMIT_PERIOD` defines the period to which authenticated requests get limited.

### BRIDGE_CACHE_TTL

- Default: 0s

When set to `d` units of time, this variable enables using cached bridge responses that are at most `d` units old. Caching is disabled by default.

Example `BRIDGE_CACHE_TTL=10s`, `BRIDGE_CACHE_TTL=1m`

### BRIDGE_RESPONSE_URL

- Default: _none_

`BRIDGE_RESPONSE_URL` defines the URL for bridges to send a response to.

Usually this will be the same as the URL/IP and port you use to connect to the Chainlink UI, such as `https://my-chainlink-node.example.com:6688`.

### HTTP_SERVER_WRITE_TIMEOUT

**ADVANCED**

Do not change this setting unless you know what you are doing.

- Default: `"10s"`

`HTTP_SERVER_WRITE_TIMEOUT` controls how long the Chainlink node's API server can hold a socket open for writing a response to an HTTP request. Sometimes, this must be increased for pprof.

### CHAINLINK_PORT

- Default: `"6688"`

Port used for the Chainlink Node API, CLI, and GUI.

### SECURE_COOKIES

- Default: `"true"`

Requires the use of secure cookies for authentication. Set to false to enable standard HTTP requests along with `CHAINLINK_TLS_PORT=0`.

### SESSION_TIMEOUT

- Default: `"15m"`

This value determines the amount of idle time to elapse before session cookies expire. This signs out GUI users from their sessions.

### UNAUTHENTICATED_RATE_LIMIT

- Default: `"5"`

`UNAUTHENTICATED_RATE_LIMIT` defines the threshold to which authenticated requests get limited. More than this many unauthenticated requests per `UNAUTHENTICATED_RATE_LIMIT_PERIOD` will be rejected.

### UNAUTHENTICATED_RATE_LIMIT_PERIOD

- Default: `"20s"`

`UNAUTHENTICATED_RATE_LIMIT_PERIOD` defines the period to which unauthenticated requests get limited.

## Web Server MFA

The Operator UI frontend now supports enabling Multi Factor Authentication via Webauthn per account. When enabled, logging in will require the account password and a hardware or OS security key such as Yubikey. To enroll, log in to the operator UI and click the circle purple profile button at the top right and then click **Register MFA Token**. Tap your hardware security key or use the OS public key management feature to enroll a key. Next time you log in, this key will be required to authenticate.

This feature must be enabled by setting the following environment variables: `MFA_RPID` and `MFA_RPORIGIN`.

### MFA_RPID

- Default: _none_

The `MFA_RPID` value should be the FQDN of where the Operator UI is served. When serving locally, the value should be `localhost`.

### MFA_RPORIGIN

- Default: _none_

The `MFA_RPORIGIN` value should be the origin URL where WebAuthn requests initiate, including scheme and port. When serving locally, the value should be `http://localhost:6688/`.

## Web Server TLS

The TLS settings below apply only if you want to enable TLS security on your Chainlink node.

### CHAINLINK_TLS_HOST

- Default: _none_

The hostname configured for TLS to be used by the Chainlink node. This is useful if you configured a domain name specific for your Chainlink node.

### CHAINLINK_TLS_PORT

- Default: `"6689"`

The port used for HTTPS connections. Set this to `0` to disable HTTPS. Disabling HTTPS also relieves Chainlink nodes of the requirement for a TLS certificate.

### CHAINLINK_TLS_REDIRECT

- Default: `"false"`

Forces TLS redirect for unencrypted connections.

### TLS_CERT_PATH

- Default: _none_

The location of the TLS certificate file. Example: `/home/$USER/.chainlink/tls/server.crt`

### TLS_KEY_PATH

- Default: _none_

The location of the TLS private key file. Example: `/home/$USER/.chainlink/tls/server.key`

## EVM/Ethereum Legacy Environment Variables

Previous Chainlink node versions supported only one chain. From v1.1.0 and up, Chainlink nodes support multiple EVM and non-EVM chains, so the way that chains and nodes are configured has changed.

The preferred way of configuring Chainlink nodes as of v1.1.0 and up is to use the API, CLI, or UI to set chain-specific configuration and create nodes.

The old way of specifying chains using environment variables is still supported, but discouraged. It works as follows:

If you set any value for `ETH_URL`, the values of `ETH_CHAIN_ID`, `ETH_URL`, `ETH_HTTP_URL` and `ETH_SECONDARY_URLS` will be used to create and update chains and nodes representing these values in the database. If an existing chain or node is found, it will be overwritten. This mode is used mainly to ease the process of upgrading. On subsequent runs (once your old settings have been written to the database) it is recommended to unset `ETH_URL` and use the API commands exclusively to administer chains and nodes.

In the future, support for the `ETH_URL` and associated environment variables might be removed, so it is recommended to use the API, CLI, or GUI instead to setup chains and nodes.

### ETH_URL

Setting this will enable "legacy eth ENV" mode, which is not compatible with multi-chain. It is better to configure settings using the API, CLI, or GUI instead.

- Default: _none_

This is the websocket address of the Ethereum client that the Chainlink node will connect to. All interaction with the Ethereum blockchain will occur through this connection.

NOTE: It is also required to set `ETH_CHAIN_ID` if you set ETH_URL.

### ETH_HTTP_URL

Only has effect if `ETH_URL` set. Otherwise, it can be set in the API, CLI, or GUI.

- Default: _none_

This should be set to the HTTP URL that points to the same ETH node as the primary. If set, the Chainlink node will automatically use HTTP mode for heavy requests, which can improve reliability.

### EVM_NODES

:::caution
Setting this environment variable will **COMPLETELY ERASE** your `evm_nodes` table on every boot and repopulate from the given data to nullify any runtime modifications. This is a temporary solution until this configuration can be defined in a file in the future.
:::

- Default: _none_

A JSON array of node specifications that allows you to configure multiple nodes or chains using an environment variable. This is not compatible with other environment variables that specify the node such as `ETH_URL` or `ETH_SECONDARY_URLS`. Set this variable using a configuration like the following example:

<!-- prettier-ignore -->
```json
EVM_NODES='
[
	{
		"name": "primary_0_1",
		"evmChainId": "0",
		"wsUrl": "ws://test1.invalid",
		"sendOnly": false
	},
	{
		"name": "primary_0_2",
		"evmChainId": "0",
		"wsUrl": "ws://test2.invalid",
		"httpUrl": "https://test3.invalid",
		"sendOnly": false
	},
	{
		"name": "primary_1337_1",
		"evmChainId": "1337",
		"wsUrl": "ws://test4.invalid",
		"httpUrl": "http://test5.invalid",
		"sendOnly": false
	},
	{
		"name": "sendonly_1337_1",
		"evmChainId": "1337",
		"httpUrl": "http://test6.invalid",
		"sendOnly": true
	},
	{
		"name": "sendonly_0_1",
		"evmChainId": "0",
		"httpUrl": "http://test7.invalid",
		"sendOnly": true
	},
	{
		"name": "primary_42_1",
		"evmChainId": "42",
		"wsUrl": "ws://test8.invalid",
		"sendOnly": false
	},
	{
		"name": "sendonly_43_1",
		"evmChainId": "43",
		"httpUrl": "http://test9.invalid",
		"sendOnly": true
	}
]
'
```

Usage of Docker requires the variable to be formatted as one line with no whitespaces and quotes wrapping it, as follows in the example:

<!-- prettier-ignore -->
```bash
EVM_NODES=[{"name":"primary_0_1","evmChainId":"0","wsUrl":"ws://test1.invalid","sendOnly":false},{"name":"primary_0_2","evmChainId":"0","wsUrl":"ws://test2.invalid","httpUrl":"https://test3.invalid","sendOnly":false},{"name":"primary_1337_1","evmChainId":"1337","wsUrl":"ws://test4.invalid","httpUrl":"http://test5.invalid","sendOnly":false}]
```

### ETH_SECONDARY_URLS

Only has effect if `ETH_URL` set. Otherwise, it can be set in the API, CLI, or GUI.

- Default: _none_

If set, transactions will also be broadcast to this secondary Ethereum node. This allows transaction broadcasting to be more robust in the face of primary Ethereum node bugs or failures.

It is recommended to set at least one secondary ETH node here that is different from your primary.

Multiple URLs can be specified as a comma-separated list e.g.

`ETH_SECONDARY_URLS=https://example.com/1,https://example.text/2,...`

## EVM/Ethereum Global Settings

This configuration is specific to EVM/Ethereum chains.

### ETH_CHAIN_ID

- Default: _none_

This environment variable specifies the default chain ID. Any job spec that has not explicitly set `EVMChainID` will connect to this default chain. If you do not have a chain in the database matching this value, any jobs that try to use it will throw an error.

### EVM_RPC_ENABLED

- Default: `"true"`

Enables connecting to real EVM RPC nodes. Disabling this can be useful in certain cases such as spinning up a Chainlink node and adding EVM-based jobs without having it actually execute anything on-chain, or for debugging to see what the node _would_ do without actually doing it.

## EVM/Ethereum Chain-specific Overrides

These configuration options act as an override, setting the value for _all_ chains.

This often doesn't make sense, e.g. `ETH_FINALITY_DEPTH` on Avalanche could be quite different from `ETH_FINALITY_DEPTH` on Ethereum mainnet.

We recommend setting this on a per-chain basis using the API, CLI, or GUI instead.

In general, Chainlink nodes contain built-in defaults for most of these settings that should work out of the box on all officially supported chains, so it is unlikely you must make any changes here.

### BALANCE_MONITOR_ENABLED

- Default: `"true"`

Enables Balance Monitor feature. This is required to track balances of keys locally and warn if it drops too low. It also enables displaying balance in the Chainlink UI and API.

### BLOCK_BACKFILL_DEPTH

- Default: `"10"`

This variable specifies the number of blocks before the current head that the log broadcaster will try to re-consume logs from, e.g. after adding a new job.

### BLOCK_BACKFILL_SKIP

- Default: `"false"`

This variable enables skipping of very long log backfills. For example, this happens in a situation when the node is started after being offline for a long time.
This might be useful on fast chains and if only recent chain events are relevant

### ETH_TX_REAPER_INTERVAL

NOTE: This overrides the setting for _all_ chains, you might want to set this on a per-chain basis using the API, CLI, or GUI instead

- Default: `"1h"`

Controls how often the ETH transaction reaper should run, used to delete old confirmed or fatally_errored transaction records from the database. Setting to `0` disables the reaper.

### ETH_TX_REAPER_THRESHOLD

- Default: `"24h"`

Represents how long any confirmed or fatally_errored `eth_tx` transactions will hang around in the database.
If the `eth_tx` is confirmed but still below `ETH_FINALITY_DEPTH`, it will not be deleted even if it was created at a time older than this value.

EXAMPLE:
With: `EthTxReaperThreshold=1h` and `EthFinalityDepth=50`
If current head is 142, any `eth_tx` confirmed in block 91 or below will be reaped as long as its `created_at` value is older than the value set for `EthTxReaperThreshold`.

Setting to `0` disables the reaper.

### ETH_TX_RESEND_AFTER_THRESHOLD

NOTE: This overrides the setting for _all_ chains, you might want to set this on a per-chain basis using the API, CLI, or GUI instead.

- Default: _automatically set based on Chain ID, typically 1m_

Controls how long the `ethResender` will wait before re-sending the latest `eth_tx_attempt`. This is designed a as a fallback to protect against the ETH nodes dropping transactions (which has been anecdotally observed to happen), networking issues, or transactions being ejected from the mempool.

Setting to `0` disables the resender.

### ETH_FINALITY_DEPTH

- Default: _automatically set based on Chain ID, typically 50_

The number of blocks after which an Ethereum transaction is considered "final".

`ETH_FINALITY_DEPTH` determines how deeply we look back to ensure that transactions are confirmed onto the longest chain. There is not a large performance penalty to setting this relatively high (on the order of hundreds).

It is practically limited by the number of heads we store in the database (`HEAD_TRACKER_HISTORY_DEPTH`) and should be less than this with a comfortable margin.
If a transaction is mined in a block more than this many blocks ago, and is reorged out, we will NOT retransmit this transaction and undefined behavior can occur including gaps in the nonce sequence that require manual intervention to fix. Therefore, this number represents a number of blocks we consider large enough that no re-org this deep will ever feasibly happen.

### ETH_HEAD_TRACKER_HISTORY_DEPTH

- Default: _automatically set based on Chain ID, typically 100_

Tracks the top N block numbers to keep in the `heads` database table. Note that this can easily result in MORE than N total records since in the case of re-orgs we keep multiple heads for a particular block height, and it is also scoped per chain. This number should be at least as large as `ETH_FINALITY_DEPTH`. There might be a small performance penalty to setting this to something very large (10,000+)

### ETH_HEAD_TRACKER_MAX_BUFFER_SIZE

- Default: `"3"`

The maximum number of heads that can be buffered in front of the head tracker before older heads start to be dropped. Think this setting as the maximum permitted "lag" for the head tracker before the Chainlink node starts dropping heads to keep up.

### ETH_HEAD_TRACKER_SAMPLING_INTERVAL

- Default: _automatically set based on Chain ID, typically 1s_

Head tracker sampling was introduced to handle chains with very high throughput. If this is set, the head tracker will "gap" heads and deliver a maximum of 1 head per this period.

Set to `0` to disable head tracker sampling.

### ETH_LOG_BACKFILL_BATCH_SIZE

- Default: _automatic based on Chain ID, typically 100_

Controls the batch size for calling FilterLogs when backfilling missing or recent logs.

### ETH_LOG_POLL_INTERVAL

- Default: _automatic based on Chain ID_

Defines how frequently to poll for new logs.

### ETH_RPC_DEFAULT_BATCH_SIZE

- Default: _automatic based on chain ID_

Chainlink nodes use batch mode for certain RPC calls to increase efficiency of communication with the remote ETH node. In some cases, trying to request too many items in a single batch will result in an error (e.g. due to bugs in go-ethereum, third-party provider limitations, limits inherent to the websocket channel etc). This setting controls the maximum number of items that can be requested in a single batch. Chainlink nodes use built-in conservative defaults for different chains that should work out of the box.

If you have enabled HTTP URLs for all of your ETH nodes, you can safely increase this to a larger value e.g. 100 and see significant RPC performance improvements.

### LINK_CONTRACT_ADDRESS

- Default: _automatic based on Chain ID_

The address of the LINK token contract. It is not essential to provide this, but if given, it is used for displaying the node account's LINK balance. For supported chains, this is automatically set based on the given chain ID. For unsupported chains, you must supply it yourself.

This environment variable is a global override. It is recommended instead to set this on a per-chain basis.

### MIN_INCOMING_CONFIRMATIONS

- Default: _automatic based on chain ID, typically 3_

The number of block confirmations to wait before kicking off a job run or proceeding with a task that listens to blockchain and log events. Setting this to a lower value improves node response time at the expense of occasionally submitting duplicate transactions in the event of chain re-orgs (duplicate transactions are harmless but cost some ETH).

You can override this on a per-job basis.

`MIN_INCOMING_CONFIRMATIONS=1` would kick off a job after seeing the transaction in just one block.

:::caution
The lowest value allowed here is 1, since setting to 0 would imply that logs are processed from the mempool before they are even mined into a block, which isn't possible with Chainlink's current architecture.
:::

### MIN_OUTGOING_CONFIRMATIONS

- Default: _automatic based on chain ID, typically 12_

The default minimum number of block confirmations that need to be recorded on an outgoing `ethtx` task before the run can move onto the next task.

This can be overridden on a per-task basis by setting the `MinRequiredOutgoingConfirmations` parameter.

`MIN_OUTGOING_CONFIRMATIONS=1` considers a transaction as "done" once it has been mined into one block.
`MIN_OUTGOING_CONFIRMATIONS=0` would consider a transaction as "done" even before it has been mined.

### MINIMUM_CONTRACT_PAYMENT_LINK_JUELS

:::note
This has replaced the formerly used MINIMUM_CONTRACT_PAYMENT.
:::

- Default: _automatically set based on Chain ID, typically 10000000000000 (0.00001 LINK) on all chains except Ethereum Mainnet and Sepolia where it is 100000000000000000 (0.1 LINK)._

For jobs that use the `EthTx` adapter, this is the minimum payment amount in order for the node to accept and process the job. Since there are no decimals on the EVM, the value is represented like wei.

:::note
Keep in mind, the Chainlink node currently responds with a 500,000 gas limit. Under pricing your node could mean it spends more in ETH (on gas) than it earns in LINK.
:::

### NODE_NO_NEW_HEADS_THRESHOLD

- Default: _automatically set based on Chain ID, typically "3m" (3 minutes)_

Controls how long to wait after receiving no new heads before marking the node as out-of-sync.

Set to zero to disable out-of-sync checking.

### NODE_POLL_FAILURE_THRESHOLD

- Default: _automatically set based on Chain ID, typically 3_

Indicates how many consecutive polls must fail in order to mark a node as unreachable.

Set to zero to disable poll checking.

### NODE_POLL_INTERVAL

- Default: _automatically set based on Chain ID, typically "10s" (10 seconds)_

Controls how often to poll the node to check for liveness.

Set to zero to disable poll checking.

### NODE_SELECTION_MODE

- Default: `"HighestHead"`

Controls node picking strategy. Supported values:

- `HighestHead` (default) mode picks a node having the highest reported head number among other alive nodes. When several nodes have the same latest head number, the strategy sticks to the last used node. This mode requires `NODE_NO_NEW_HEADS_THRESHOLD` to be configured, otherwise it will always use the first alive node.
- `RoundRobin` mode simply iterates among available alive nodes. This was the default behavior prior to this release.
- `TotalDifficulty` mode selects the node with the greatest total difficulty.

### NODE_SYNC_THRESHOLD

- Default: `5`

SyncThreshold controls how far a node may lag behind the best node before being marked out-of-sync. Depending on the [`NODE_SELECTION_MODE` variable](/chainlink-nodes/v1/configuration/#node_selection_mode), this represents a difference in either the number of blocks (`HighestHead`, `RoundRobin`), or the total difficulty (`TotalDifficulty`).

Set to `0` to disable this check.

## EVM Gas Controls

These settings allow you to tune your node's gas limits and pricing. In most cases, leaving these values at their defaults should give good results.

As of Chainlink node v1.1.0, it is recommended to use the API, CLI, or GUI to configure gas controls because you might want to use different settings for different chains. Setting the environment variable typically overrides the setting for all chains.

### Configuring your ETH node

Your ETH node might need some configuration tweaks to make it fully compatible with Chainlink nodes depending on your configuration.

#### go-ethereum

WARNING: By default, go-ethereum will reject transactions that exceed it's built-in RPC gas or txfee caps. Chainlink nodes will fatally error transactions if this happens which means if you ever exceed the caps your node will miss transactions.

You should at a bare minimum disable the default RPC gas and txfee caps on your ETH node. This can be done in the TOML file as seen below, or by running go-ethereum with the command line arguments: `--rpc.gascap=0 --rpc.txfeecap=0`.

It is also recommended to configure go-ethereum properly before increasing `ETH_MAX_IN_FLIGHT_TRANSACTIONS` to ensure all in-flight transactions are maintained in the mempool.

Relevant settings for geth and forks (such as BSC).

<!-- prettier-ignore -->
```toml
[Eth]
RPCGasCap = 0 # it is recommended to disable both gas and txfee cap
RPCTxFeeCap = 0.0
[Eth.TxPool]
Locals = ["0xYourNodeAddress1", "0xYourNodeAddress2"]  # Add your node addresses here
NoLocals = false # Disabled by default but might as well make sure
Journal = "transactions.rlp" # Make sure you set a journal file
Rejournal = 3600000000000 # Default 1h, it might make sense to reduce this to e.g. 5m
PriceBump = 10 # Must be set less than or equal to Chainlink's ETH_GAS_BUMP_PERCENT
AccountSlots = 16 # Highly recommended to increase this, must be greater than or equal to Chainlink's ETH_MAX_IN_FLIGHT_TRANSACTIONS setting
GlobalSlots = 4096 # Increase this as necessary
AccountQueue = 64 # Increase this as necessary
GlobalQueue = 1024 # Increase this as necessary
Lifetime = 10800000000000 # Default 3h, this is probably ok, you might even consider reducing it
```

### EVM_EIP1559_DYNAMIC_FEES

- Default: _automatic based on chain ID_

Forces EIP-1559 transaction mode for all chains. Enabling EIP-1559 mode can help reduce gas costs on chains that support it. This is supported only on official Ethereum mainnet and testnets. It is not recommended to enable this setting on Polygon because the EIP-1559 fee market appears to be broken on all Polygon chains and EIP-1559 transactions are less likely to be included than legacy transactions.

#### Technical details

Chainlink nodes include experimental support for submitting transactions using type 0x2 (EIP-1559) envelope.

EIP-1559 mode is enabled by default on the Ethereum Mainnet, but can be enabled on a per-chain basis or globally.

This might help to save gas on spikes. Chainlink nodes should react faster on the upleg and avoid overpaying on the downleg. It might also be possible to set `BLOCK_HISTORY_ESTIMATOR_BATCH_SIZE` to a smaller value such as 12 or even 6 because tip cap should be a more consistent indicator of inclusion time than total gas price. This would make Chainlink nodes more responsive and should reduce response time variance. Some experimentation is required to find optimum settings.

To enable globally, set `EVM_EIP1559_DYNAMIC_FEES=true`. Set with caution, if you set this on a chain that does not actually support EIP-1559 your node will be broken.

In EIP-1559 mode, the total price for the transaction is the minimum of base fee + tip cap and fee cap. More information can be found on the [official EIP](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1559.md).

Chainlink's implementation of EIP-1559 works as follows:

If you are using FixedPriceEstimator:

- With gas bumping disabled, it will submit all transactions with `feecap=ETH_MAX_GAS_PRICE_WEI` and `tipcap=EVM_GAS_TIP_CAP_DEFAULT`
- With gas bumping enabled, it will submit all transactions initially with `feecap=EVM_GAS_FEE_CAP_DEFAULT` and `tipcap=EVM_GAS_TIP_CAP_DEFAULT`.

If you are using BlockHistoryEstimator (default for most chains):

- With gas bumping disabled, it will submit all transactions with `feecap=ETH_MAX_GAS_PRICE_WEI` and `tipcap=<calculated using past blocks>`
- With gas bumping enabled (default for most chains) it will submit all transactions initially with `feecap=current block base fee * (1.125 ^ N)` where N is configurable by setting BLOCK_HISTORY_ESTIMATOR_EIP1559_FEE_CAP_BUFFER_BLOCKS but defaults to `gas bump threshold+1` and `tipcap=<calculated using past blocks>`

Bumping works as follows:

- Increase tipcap by `max(tipcap * (1 + ETH_GAS_BUMP_PERCENT), tipcap + ETH_GAS_BUMP_WEI)`
- Increase feecap by `max(feecap * (1 + ETH_GAS_BUMP_PERCENT), feecap + ETH_GAS_BUMP_WEI)`

A quick note on terminology - Chainlink nodes use the same terms used internally by go-ethereum source code to describe various prices. This is not the same as the externally used terms. For reference:

- Base Fee Per Gas = BaseFeePerGas
- Max Fee Per Gas = FeeCap
- Max Priority Fee Per Gas = TipCap

In EIP-1559 mode, the following changes occur to how configuration works:

- All new transactions will be sent as type 0x2 transactions specifying a TipCap and FeeCap. Be aware that existing pending legacy transactions will continue to be gas bumped in legacy mode.
- `BlockHistoryEstimator` will apply its calculations (gas percentile etc) to the TipCap and this value will be used for new transactions (GasPrice will be ignored)
- `FixedPriceEstimator` will use `EVM_GAS_TIP_CAP_DEFAULT` instead of `ETH_GAS_PRICE_DEFAULT` for the tip cap
- `FixedPriceEstimator` will use `EVM_GAS_FEE_CAP_DEFAULT` instaed of `ETH_GAS_PRICE_DEFAULT` for the fee cap
- `ETH_MIN_GAS_PRICE_WEI` is ignored for new transactions and `EVM_GAS_TIP_CAP_MINIMUM` is used instead (default 0)
- `ETH_MAX_GAS_PRICE_WEI` still represents that absolute upper limit that Chainlink will ever spend (total) on a single tx
- `KEEPER_GAS_PRICE_BUFFER_PERCENT` is ignored in EIP-1559 mode and `KEEPER_TIP_CAP_BUFFER_PERCENT` is used instead

### ETH_GAS_BUMP_PERCENT

- Default: _automatic based on chain ID_

The percentage by which to bump gas on a transaction that has exceeded `ETH_GAS_BUMP_THRESHOLD`. The larger of `ETH_GAS_BUMP_PERCENT` and `ETH_GAS_BUMP_WEI` is taken for gas bumps.

### ETH_GAS_BUMP_THRESHOLD

- Default: _automatic based on chain ID_

Chainlink nodes can be configured to automatically bump gas on transactions that have been stuck waiting in the mempool for at least this many blocks. Set to 0 to disable gas bumping completely.

### ETH_GAS_BUMP_TX_DEPTH

- Default: `"10"`

The number of transactions to gas bump starting from oldest. Set to 0 for no limit (i.e. bump all).

### ETH_GAS_BUMP_WEI

- Default: _automatic based on chain ID_

The minimum fixed amount of wei by which gas is bumped on each transaction attempt.

### EVM_GAS_FEE_CAP_DEFAULT

- Default: _automatic based on chain ID_

If EIP1559 mode is enabled, and FixedPrice gas estimator is used, this env var controls the fixed initial fee cap.

### ETH_GAS_LIMIT_DEFAULT

- Default: _automatically set based on Chain ID, typically 500000_

The default gas limit for outgoing transactions. This should not need to be changed in most cases.
Some job types, such as Keeper jobs, might set their own gas limit unrelated to this value.

### ETH_GAS_LIMIT_MAX

- Default: _automatically set based on Chain ID, typically 500000_

The maxium for gas limits estimated by the `Arbitrum` `GAS_ESTIMATOR_MODE`. This should not need to be changed in most cases.

### ETH_GAS_LIMIT_MULTIPLIER

- Default: `"1.0"`

A factor by which a transaction's GasLimit is multiplied before transmission. So if the value is 1.1, and the GasLimit for a transaction is 10, 10% will be added before transmission.

This factor is always applied, so includes Optimism L2 transactions which uses a default gas limit of 1 and is also applied to EthGasLimitDefault.

### ETH_GAS_LIMIT_TRANSFER

- Default: _automatically set based on Chain ID, typically 21000_

The gas limit used for an ordinary ETH transfer.

### ETH_GAS_PRICE_DEFAULT

(Only applies to legacy transactions)

- Default: _automatic based on chain ID_

The default gas price to use when submitting transactions to the blockchain. Will be overridden by the built-in `BlockHistoryEstimator` if enabled, and might be increased if gas bumping is enabled.

Can be used with the `chainlink setgasprice` to be updated while the node is still running.

### EVM_GAS_TIP_CAP_DEFAULT

(Only applies to EIP-1559 transactions)

- Default: _automatic based on chain ID_

The default gas tip to use when submitting transactions to the blockchain. Will be overridden by the built-in `BlockHistoryEstimator` if enabled, and might be increased if gas bumping is enabled.

### EVM_GAS_TIP_CAP_MINIMUM

(Only applies to EIP-1559 transactions)

- Default: _automatic based on chain ID_

The minimum gas tip to use when submitting transactions to the blockchain.

### ETH_MAX_GAS_PRICE_WEI

- Default: _automatic based on chain ID_

Chainlink nodes will never pay more than this for a transaction.

### ETH_MAX_IN_FLIGHT_TRANSACTIONS

- Default: `"16"`

Controls how many transactions are allowed to be "in-flight" i.e. broadcast but unconfirmed at any one time. You can consider this a form of transaction throttling.

The default is set conservatively at 16 because this is a pessimistic minimum that geth will hold without evicting local transactions. If your node is falling behind and you need higher throughput, you can increase this setting, but you MUST make sure that your ETH node is configured properly otherwise you can get nonce gapped and your node will get stuck.

0 value disables the limit. Use with caution.

### ETH_MAX_QUEUED_TRANSACTIONS

- Default: _automatically set based on Chain ID, typically 250_

The maximum number of unbroadcast transactions per key that are allowed to be enqueued before jobs will start failing and rejecting send of any further transactions. This represents a sanity limit and generally indicates a problem with your ETH node (transactions are not getting mined).

Do NOT blindly increase this value thinking it will fix things if you start hitting this limit because transactions are not getting mined, you will instead only make things worse.

In deployments with very high burst rates, or on chains with large re-orgs, you _may_ consider increasing this.

0 value disables any limit on queue size. Use with caution.

### ETH_MIN_GAS_PRICE_WEI

(Only applies to legacy transactions)

- Default: _automatic based on chain ID_

Chainlink nodes will never pay less than this for a transaction.

It is possible to force the Chainlink node to use a fixed gas price by setting a combination of these, e.g.

```text
EVM_EIP1559_DYNAMIC_FEES=false
ETH_MAX_GAS_PRICE_WEI=100
ETH_MIN_GAS_PRICE_WEI=100
ETH_GAS_PRICE_DEFAULT=100
ETH_GAS_BUMP_THRESHOLD=0
GAS_ESTIMATOR_MODE="FixedPrice"
```

### ETH_GAS_LIMIT_OCR_JOB_TYPE

- Default: _none_

Overrides the [default gas limit](#eth_gas_limit_default) for OCR jobs. This environment variable does not override task-specific or job-specific `gasLimit` parameters or attributes.

### ETH_GAS_LIMIT_DR_JOB_TYPE

- Default: _none_

Overrides the [default gas limit](#eth_gas_limit_default) for direct request jobs. This environment variable does not override task-specific or job-specific `gasLimit` parameters or attributes.

### ETH_GAS_LIMIT_VRF_JOB_TYPE

- Default: _none_

Overrides the [default gas limit](#eth_gas_limit_default) for VRF jobs. This environment variable does not override task-specific or job-specific `gasLimit` parameters or attributes.

### ETH_GAS_LIMIT_FM_JOB_TYPE

- Default: _none_

Overrides the [default gas limit](#eth_gas_limit_default) for Flux Monitor jobs. This environment variable does not override task-specific or job-specific `gasLimit` parameters or attributes.

### ETH_GAS_LIMIT_KEEPER_JOB_TYPE

- Default: _none_

Overrides the [default gas limit](#eth_gas_limit_default) for Keeper jobs. This environment variable does not override task-specific or job-specific `gasLimit` parameters or attributes.

### ETH_NONCE_AUTO_SYNC

- Default: `"false"`

Chainlink nodes will automatically try to sync its local nonce with the remote chain on startup and fast forward if necessary. This is almost always safe but can be disabled in exceptional cases by setting this value to false.

### ETH_USE_FORWARDERS

- Default: `"false"`

Enables or disables sending transactions through forwarder contracts.

## EVM/Ethereum Gas Price Estimation

These settings allow you to configure how your node calculates gas prices. In most cases, leaving these values at their defaults should give good results.

As of Chainlink node v1.1.0, it is recommended to use the API, CLI, or GUI to configure gas controls because you might want to use different settings for different chains. Setting the environment variable typically overrides the setting for all chains.

Chainlink nodes decide what gas price to use using an `Estimator`. It ships with several simple and battle-hardened built-in estimators that should work well for almost all use-cases. Note that estimators will change their behaviour slightly depending on if you are in EIP-1559 mode or not.

You can also use your own estimator for gas price by selecting the `FixedPrice` estimator and using the exposed API to set the price.

An important point to note is that the Chainlink node does _not_ ship with built-in support for go-ethereum's `estimateGas` call. This is for several reasons, including security and reliability. We have found empirically that it is not generally safe to rely on the remote ETH node's idea of what gas price should be.

### GAS_ESTIMATOR_MODE

- Default: _automatic, based on chain ID_

Controls what type of gas estimator is used.

- `FixedPrice` uses static configured values for gas price (can be set via API call).
- `BlockHistory` dynamically adjusts default gas price based on heuristics from mined blocks.
- `Optimism2`/`L2Suggested` is a special mode only for use with Optimism and Metis blockchains. This mode will use the gas price suggested by the rpc endpoint via `eth_gasPrice`.
- `Arbitrum` is a special mode only for use with Arbitrum blockchains. It uses the suggested gas price (up to `ETH_MAX_GAS_PRICE_WEI`, with `1000 gwei` default) as well as an estimated gas limit (up to `ETH_GAS_LIMIT_MAX`, with `1,000,000,000` default).

### BLOCK_HISTORY_ESTIMATOR_BATCH_SIZE

- Default: _automatic, based on chain ID, typically 4_

Sets the maximum number of blocks to fetch in one batch in the block history estimator.
If the `BLOCK_HISTORY_ESTIMATOR_BATCH_SIZE` environment variable is set to 0, it defaults to ETH_RPC_DEFAULT_BATCH_SIZE.

### BLOCK_HISTORY_ESTIMATOR_BLOCK_HISTORY_SIZE

- Default: _automatic, based on chain ID_

Controls the number of past blocks to keep in memory to use as a basis for calculating a percentile gas price.

### BLOCK_HISTORY_ESTIMATOR_BLOCK_DELAY

- Default: _automatic, based on chain ID_

Controls the number of blocks that the block history estimator trails behind head.
For example, if this is set to 3, and we receive block 10, block history estimator will fetch block 7.

CAUTION: You might be tempted to set this to 0 to use the latest possible
block, but it is possible to receive a head BEFORE that block is actually
available from the connected node via RPC, due to race conditions in the code of the remote ETH node. In this case you will get false
"zero" blocks that are missing transactions.

### BLOCK_HISTORY_ESTIMATOR_EIP1559_FEE_CAP_BUFFER_BLOCKS

**ADVANCED**

- Default: _gas bump threshold + 1 block_

If EIP1559 mode is enabled, this optional env var controls the buffer blocks to add to the current base fee when sending a transaction. By default, the gas bumping threshold + 1 block is used. It is not recommended to change this unless you know what you are doing.

### BLOCK_HISTORY_ESTIMATOR_TRANSACTION_PERCENTILE

- Default: `"60"`

Must be in range 0-100.

Only has an effect if gas updater is enabled. Specifies percentile gas price to choose. E.g. if the block history contains four transactions with gas prices `[100, 200, 300, 400]` then picking 25 for this number will give a value of 200. If the calculated gas price is higher than `ETH_GAS_PRICE_DEFAULT` then the higher price will be used as the base price for new transactions.

Think of this number as an indicator of how aggressive you want your node to price its transactions.

Setting this number higher will cause the Chainlink node to select higher gas prices.

Setting it lower will tend to set lower gas prices.

## EVM/Ethereum Transaction Simulation

Chainlink nodes support transaction simulation for certain types of job. When this is enabled, transactions will be simulated using `eth_call` before initial send. If the transaction would revert, the transaction is marked as an error without being broadcast, potentially avoiding an expensive on-chain revert.

This can add a tiny bit of latency with an upper bound of 2s, but generally much shorter under good conditions. This will add marginally more load to the ETH client, because it adds an extra call for every transaction sent. However, it might help to save gas in some cases especially during periods of high demand by avoiding unnecessary reverts due to outdated round etc.

This option is EXPERIMENTAL and disabled by default.

To enable for FM or OCR:

`FM_SIMULATE_TRANSACTIONs=true`
`OCR_SIMULATE_TRANSACTIONS=true`

To enable in the pipeline, use the `simulate=true` option like so:

```toml
submit [type=ethtx to="0xDeadDeadDeadDeadDeadDeadDeadDead" data="0xDead" simulate=true]
```

Use at your own risk.

#### FM_SIMULATE_TRANSACTIONS

NOTE: This overrides the setting for _all_ chains, it is not currently possible to configure this on a per-chain basis.

- Default: `"false"`

`FM_SIMULATE_TRANSACTIONS` allows to enable transaction simulation for Flux Monitor.

#### OCR_SIMULATE_TRANSACTIONS

NOTE: This overrides the setting for _all_ chains, it is not currently possible to configure this on a per-chain basis.

- Default: `"false"`

`OCR_SIMULATE_TRANSACTIONS` allows to enable transaction simulation for OCR.

## Job Pipeline and tasks

### DEFAULT_HTTP_LIMIT

- Default: `"32768"`

`DEFAULT_HTTP_LIMIT` defines the maximum number of bytes for HTTP requests and responses made by `http` and `bridge` adapters.

### DEFAULT_HTTP_TIMEOUT

- Default: `"15s"`

`DEFAULT_HTTP_TIMEOUT` defines the default timeout for HTTP requests made by `http` and `bridge` adapters.

### FEATURE_EXTERNAL_INITIATORS

- Default: `"false"`

Enables the External Initiator feature. If disabled, `webhook` jobs can ONLY be initiated by a logged-in user. If enabled, `webhook` jobs can be initiated by a whitelisted external initiator.

### JOB_PIPELINE_MAX_RUN_DURATION

- Default: `"10m"`

`JOB_PIPELINE_MAX_RUN_DURATION` is the maximum time that a single job run might take. If it takes longer, it will exit early and be marked errored. If set to zero, disables the time limit completely.

### JOB_PIPELINE_MAX_SUCCESSFUL_RUNS

This option is not supported as an environment variable. Use `JobPipeline.MaxSuccessfulRuns` in the config file instead. See the [CONFIG.md](https://github.com/smartcontractkit/chainlink/blob/v1.12.0/docs/CONFIG.md) reference for details.

### JOB_PIPELINE_REAPER_INTERVAL

- Default: `"1h"`

In order to keep database size manageable, Chainlink nodes will run a reaper that deletes completed job runs older than a certain threshold age. `JOB_PIPELINE_REAPER_INTERVAL` controls how often the job pipeline reaper will run.

Set to `0` to disable the periodic reaper.

### JOB_PIPELINE_REAPER_THRESHOLD

- Default: `"24h"`

`JOB_PIPELINE_REAPER_THRESHOLD` determines the age limit for job runs. Completed job runs older than this will be automatically purged from the database.

### JOB_PIPELINE_RESULT_WRITE_QUEUE_DEPTH

- Default: `"100"`

Some jobs write their results asynchronously for performance reasons such as OCR. `JOB_PIPELINE_RESULT_WRITE_QUEUE_DEPTH` controls how many writes will be buffered before subsequent writes are dropped.

Do not change this setting unless you know what you are doing.

## OCR

This section applies only if you are running off-chain reporting jobs.

### FEATURE_OFFCHAIN_REPORTING

- Default: `"false"`

Set to `true` to enable OCR jobs.

### OCR_KEY_BUNDLE_ID

- Default: _none_

`OCR_KEY_BUNDLE_ID` is the default key bundle ID to use for OCR jobs. If you have an OCR job that does not explicitly specify a key bundle ID, it will fall back to this value.

### OCR_MONITORING_ENDPOINT

- Default: _none_

Optional URL of OCR monitoring endpoint.

### OCR_TRANSMITTER_ADDRESS

- Default: _none_

`OCR_TRANSMITTER_ADDRESS` is the default sending address to use for OCR. If you have an OCR job that does not explicitly specify a transmitter address, it will fall back to this value.

### P2P_NETWORKING_STACK

- Default: `"V1"`

OCR supports multiple networking stacks. `P2P_NETWORKING_STACK` chooses which stack to use. Possible values are:

- `V1`
- `V1V2` - Runs both stacks simultaneously. For each link with another peer, V2 networking will be preferred. If V2 does not work, the link will automatically fall back to V1. If V2 starts working again later, it will automatically be prefered again. This is useful for migrating networks without downtime. Note that the two networking stacks _must not_ be configured to bind to the same IP/port.
- `V2`

All nodes in the OCR network should share the same networking stack. The `V1` stack is deprecated and is being phased out. Do not use it for new deployments. Expect the default value of this variable to change to `V2` in the future.

#### P2P_PEER_ID

- Default: _none_

This environment variable is used for both Networking Stack V1 and V2.

The default peer ID to use for OCR jobs. If unspecified, uses the first available peer ID.
Example: `P2P_PEER_ID=12D3KooWMHMRLQkgPbFSYHwD3NBuwtS1AmxhvKVUrcfyaGDASR4U`

### Networking Stack V1

:::caution
Do not set environment variables for Networking Stack v1 if you are using [Networking Stack V2](#networking-stack-v2).
:::

#### P2P_ANNOUNCE_IP

- Default: _none_

Should be set as the externally reachable IP address of the Chainlink node.
Example: `P2P_ANNOUNCE_IP=1.2.3.4`

#### P2P_ANNOUNCE_PORT

- Default: _none_

Should be set as the externally reachable port of the Chainlink node.
Example: `P2P_ANNOUNCE_PORT=1337`

#### P2P_BOOTSTRAP_PEERS

- Default: _none_

Default set of bootstrap peers.
Example: `P2P_BOOTSTRAP_PEERS=/dns4/example.com/tcp/1337/p2p/12D3KooWMHMRLQkgPbFSYHwD3NBuwtS1AmxhvKVUrcfyaGDASR4U /ip4/1.2.3.4/tcp/9999/p2p/12D3KooWLZ9uTC3MrvKfDpGju6RAQubiMDL7CuJcAgDRTYP7fh7R`

#### P2P_LISTEN_IP

- Default: `"0.0.0.0"`

The default IP address to bind to.

#### P2P_LISTEN_PORT

- Default: _none_

The port to listen on. If left blank, the node randomly selects a different port each time it boots. It is highly recommended to set this to a static value to avoid network instability.

#### P2P_PEER_ID

- Default: _none_

This environment variable is used for both Networking Stack V1 and V2.

The default peer ID to use for OCR jobs. If unspecified, uses the first available peer ID.
Example: `P2P_PEER_ID=12D3KooWMHMRLQkgPbFSYHwD3NBuwtS1AmxhvKVUrcfyaGDASR4U`

### Networking Stack V2

:::caution
If using the Networking Stack V2, you must unset the following [Networking Stack V1](#networking-stack-v1) configuration variables:

- [P2P_ANNOUNCE_IP](#p2p_announce_ip)
- [P2P_ANNOUNCE_PORT](#p2p_announce_port)
- [P2P_BOOTSTRAP_PEERS](#p2p_bootstrap_peers)
- [P2P_LISTEN_IP](#p2p_listen_ip)
- [P2P_LISTEN_PORT](#p2p_listen_port)

[`P2P_PEER_ID`](#p2p_peer_id) is used for both Networking Stack V1 and V2.
:::

The Networking Stack V2 uses TCP, any ports mentioned in this section refer to TCP ports.

#### P2PV2_ANNOUNCE_ADDRESSES

- Default: _none_

`P2PV2_ANNOUNCE_ADDRESSES` contains the addresses the node will advertise for peer discovery in host:port form as accepted by the TCP version of Go's [`net.Dial`](https://pkg.go.dev/net#Dial). The addresses should be reachable by other nodes on the network. When attempting to connect to another node, a node will attempt to dial all of the other node's `P2PV2_ANNOUNCE_ADDRESSES` in round-robin fashion.
Example: `P2PV2_ANNOUNCE_ADDRESSES=1.2.3.4:9999 [a52d:0:a88:1274::abcd]:1337`

#### P2PV2_BOOTSTRAPPERS

- Default: _none_

`P2PV2_BOOTSTRAPPERS` returns the default bootstrapper peers for libocr's v2 networking stack.
Example: `P2PV2_BOOTSTRAPPERS=12D3KooWMHMRLQkgPbFSYHwD3NBuwtS1AmxhvKVUrcfyaGDASR4U@1.2.3.4:9999 12D3KooWLZ9uTC3MrvKfDpGju6RAQubiMDL7CuJcAgDRTYP7fh7R@[a52d:0:a88:1274::abcd]:1337 12D3KooWM55u5Swtpw9r8aFLQHEtw7HR4t44GdNs654ej5gRs2Dh@example.com:1234`

Oracle nodes typically only know each other's PeerIDs, but not their hostnames, IP addresses, or ports.
Bootstrappers are special nodes that help other nodes discover each other's `P2PV2_ANNOUNCE_ADDRESSES` so they can communicate.
Nodes continuously attempt to connect to bootstrappers configured in `P2PV2_BOOTSTRAPPERS`.
When a node wants to connect to another node (which it knows only by PeerID, but not by address), it discovers the other node's `P2PV2_ANNOUNCE_ADDRESSES` from communications received from its `P2PV2_BOOTSTRAPPERS` or other discovered nodes.
To facilitate discovery, nodes will regularly broadcast signed announcements containing their PeerID and `P2PV2_ANNOUNCE_ADDRESSES`.

#### P2PV2_LISTEN_ADDRESSES

- Default: _none_

`P2PV2_LISTEN_ADDRESSES` contains the addresses the peer will listen to on the network in `host:port` form as accepted by `net.Listen()`, but the host and port must be fully specified and cannot be empty. You can specify `0.0.0.0` (IPv4) or `::` (IPv6) to listen on all interfaces, but that is not recommended.

Example: `P2PV2_LISTEN_ADDRESSES=1.2.3.4:9999 [a52d:0:a88:1274::abcd]:1337`

## Keeper

These environment variables are used specificly for Chainlink Keepers. For most Chainlink Nodes, leave these values at their defaults and do not configure these environment variables.

### KEEPER_GAS_PRICE_BUFFER_PERCENT

:::caution
Do not change this setting unless you know what you are doing.
:::

- Default: `"20"`

`KEEPER_GAS_PRICE_BUFFER_PERCENT` adds the specified percentage to the gas price used for checking whether to perform an upkeep. Only applies in legacy mode (EIP-1559 off).

### KEEPER_GAS_TIP_CAP_BUFFER_PERCENT

:::caution
Do not change this setting unless you know what you are doing.
:::

- Default: `"20"`

`KEEPER_GAS_TIP_CAP_BUFFER_PERCENT` adds the specified percentage to the gas price used for checking whether to perform an upkeep. Only applies in EIP-1559 mode.

### KEEPER_BASE_FEE_BUFFER_PERCENT

:::caution
Do not change this setting unless you know what you are doing.
:::

- Default: `"20"`

Adds the specified percentage to the base fee used for checking whether to perform an upkeep. Applies only in EIP-1559 mode.

### KEEPER_MAXIMUM_GRACE_PERIOD

:::caution
Do not change this setting unless you know what you are doing.
:::

- Default: `"100"`

The maximum number of blocks that a keeper will wait after performing an upkeep before it resumes checking that upkeep

### KEEPER_REGISTRY_CHECK_GAS_OVERHEAD

:::caution
Do not change this setting unless you know what you are doing.
:::

- Default: `"200000"`

The amount of extra gas to provide checkUpkeep() calls to account for the gas consumed by the keeper registry.

### KEEPER_REGISTRY_PERFORM_GAS_OVERHEAD

:::caution
Do not change this setting unless you know what you are doing.
:::

- Default: `"300000"`

The amount of extra gas to provide performUpkeep() calls to account for the gas consumed by the keeper registry

### KEEPER_REGISTRY_SYNC_INTERVAL

:::caution
Do not change this setting unless you know what you are doing.
:::

- Default: `"30m"`

The interval in which the RegistrySynchronizer performs a full sync of the keeper registry contract it is tracking.

### KEEPER_REGISTRY_SYNC_UPKEEP_QUEUE_SIZE

:::caution
Do not change this setting unless you know what you are doing.
:::

- Default: `"10"`

`KEEPER_REGISTRY_SYNC_UPKEEP_QUEUE_SIZE` represents the maximum number of upkeeps that can be synced in parallel.

### KEEPER_TURN_LOOK_BACK

:::caution
Do not change this setting unless you know what you are doing.
:::

- Default: `"1000"`

The number of blocks in the past to look back when getting a block for a turn.

## CLI Client

The environment variables in this section apply only when running CLI commands that connect to a remote running instance of a Chainlink node.

### ADMIN_CREDENTIALS_FILE

:::caution[Deprecated]
This environment variable is deprecated and will be removed in a future release. Use the `--admin-credentials-file FILE` CLI argument instead.
:::

- Default: `$ROOT/apicredentials`

`ADMIN_CREDENTIALS_FILE` optionally points to a text file containing admin credentials for logging in. It is useful for running client CLI commands and has no effect when passed to a running node.

The file should contain two lines, the first line is the username and second line is the password.
e.g.

```text
myusername@example.com
mysecurepassw0rd
```

### CLIENT_NODE_URL

:::caution[Deprecated]
This environment variable is deprecated and will be removed in a future release. Use the `--remote-node-url URL` CLI argument instead.
:::

- Default: `"http://localhost:6688"`

This is the URL that you will use to interact with the node, including the GUI. Use this URL to connect to the GUI or to run commands remotely using the Chainlink CLI.

### INSECURE_SKIP_VERIFY

:::caution[Deprecated]
This environment variable is deprecated and will be removed in a future release. Use the `--insecure-skip-verify` CLI argument instead.
:::

- Default: `"false"`

`INSECURE_SKIP_VERIFY` disables SSL certificate verification when connection to a Chainlink node using the remote client. For example, when executing most remote commands in the CLI. This is mostly useful for people who want to use TLS on localhost.

It is not recommended to change this unless you know what you are doing.

## Notes on setting environment variables

:::note
Some environment variables require a duration. A duration string is a possibly signed sequence of decimal numbers, each with optional fraction and a unit suffix, such as "300ms", "-1.5h" or "2h45m". Valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h". Some examples:
:::

`10ms`
`1h15m`
`42m30s`

:::note
Some configuration variables require a file size. A file size string is an unsigned integer (123) or a float (12.3) followed by a unit suffix. Valid file size units are "b", "kb", "mb", "gb", and "tb". If the unit is omitted, it is assumed to be "b" (bytes). Capitalization does not matter. Some examples:
:::

`123gb`
`1.2TB`
`12345`
