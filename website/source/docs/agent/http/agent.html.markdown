---
layout: "docs"
page_title: "Agent (HTTP)"
sidebar_current: "docs-agent-http-agent"
description: >
  The Agent endpoints are used to interact with the local Consul agent.
---

# Agent HTTP Endpoint

The Agent endpoints are used to interact with the local Consul agent. Usually,
services and checks are registered with an agent which then takes on the
burden of keeping that data synchronized with the cluster. For example, the
agent registers services and checks with the Catalog and performs
[anti-entropy](/docs/internals/anti-entropy.html) to recover from outages.

The following endpoints are supported:

* [`/v1/agent/checks`](#agent_checks) : Returns the checks the local agent is managing
* [`/v1/agent/services`](#agent_services) : Returns the services the local agent is managing
* [`/v1/agent/members`](#agent_members) : Returns the members as seen by the local serf agent
* [`/v1/agent/self`](#agent_self) : Returns the local node configuration
* [`/v1/agent/reload`](#agent_reload) : Causes the local agent to reload its configuration
* [`/v1/agent/maintenance`](#agent_maintenance) : Manages node maintenance mode
* [`/v1/agent/monitor`](#agent_monitor) : Streams logs from the local agent
* [`/v1/agent/join/<address>`](#agent_join) : Triggers the local agent to join a node
* [`/v1/agent/leave`](#agent_leave): Triggers the local agent to gracefully shutdown and leave the cluster
* [`/v1/agent/force-leave/<node>`](#agent_force_leave): Forces removal of a node
* [`/v1/agent/check/register`](#agent_check_register) : Registers a new local check
* [`/v1/agent/check/deregister/<checkID>`](#agent_check_deregister) : Deregisters a local check
* [`/v1/agent/check/pass/<checkID>`](#agent_check_pass) : Marks a local check as passing
* [`/v1/agent/check/warn/<checkID>`](#agent_check_warn) : Marks a local check as warning
* [`/v1/agent/check/fail/<checkID>`](#agent_check_fail) : Marks a local check as critical
* [`/v1/agent/check/update/<checkID>`](#agent_check_update) : Updates a local check
* [`/v1/agent/service/register`](#agent_service_register) : Registers a new local service
* [`/v1/agent/service/deregister/<serviceID>`](#agent_service_deregister) : Deregisters a local service
* [`/v1/agent/service/maintenance/<serviceID>`](#agent_service_maintenance) : Manages service maintenance mode

### <a name="agent_checks"></a> /v1/agent/checks

This endpoint is used to return all the checks that are registered with
the local agent. These checks were either provided through configuration files
or added dynamically using the HTTP API. It is important to note that the checks
known by the agent may be different from those reported by the Catalog. This is usually
due to changes being made while there is no leader elected. The agent performs active
[anti-entropy](/docs/internals/anti-entropy.html), so in most situations everything will
be in sync within a few seconds.

This endpoint is hit with a `GET` and returns a JSON body like this:

```javascript
{
  "service:redis": {
    "Node": "foobar",
    "CheckID": "service:redis",
    "Name": "Service 'redis' check",
    "Status": "passing",
    "Notes": "",
    "Output": "",
    "ServiceID": "redis",
    "ServiceName": "redis"
  }
}
```

### <a name="agent_services"></a> /v1/agent/services

This endpoint is used to return all the services that are registered with
the local agent. These services were either provided through configuration files
or added dynamically using the HTTP API. It is important to note that the services
known by the agent may be different from those reported by the Catalog. This is usually
due to changes being made while there is no leader elected. The agent performs active
[anti-entropy](/docs/internals/anti-entropy.html), so in most situations everything will
be in sync within a few seconds.

This endpoint is hit with a `GET` and returns a JSON body like this:

```javascript
{
  "redis": {
    "ID": "redis",
    "Service": "redis",
    "Tags": null,
    "Address": "",
    "Port": 8000
  }
}
```

### <a name="agent_members"></a> /v1/agent/members

This endpoint is used to return the members the agent sees in the
cluster gossip pool. Due to the nature of gossip, this is eventually consistent: the
results may differ by agent. The strongly consistent view of nodes is
instead provided by `/v1/catalog/nodes`.

For agents running in server mode, providing a `?wan=1` query parameter returns
the list of WAN members instead of the LAN members returned by default.

This endpoint is hit with a `GET` and returns a JSON body like:

```javascript
[
  {
    "Name": "foobar",
    "Addr": "10.1.10.12",
    "Port": 8301,
    "Tags": {
      "bootstrap": "1",
      "dc": "dc1",
      "port": "8300",
      "role": "consul"
    },
    "Status": 1,
    "ProtocolMin": 1,
    "ProtocolMax": 2,
    "ProtocolCur": 2,
    "DelegateMin": 1,
    "DelegateMax": 3,
    "DelegateCur": 3
  }
]
```

### <a name="agent_self"></a> /v1/agent/self

This endpoint is used to return the configuration and member information of the local agent under the `Config` key.

Consul 0.7.0 and later also includes a snapshot of various operating statistics under the `Stats` key. These statistics are intended to help human operators for debugging and may change over time, so this part of the interface should not be consumed programmatically.

It returns a JSON body like this:

```javascript
{
  "Config": {
    "Bootstrap": true,
    "Server": true,
    "Datacenter": "dc1",
    "DataDir": "/tmp/consul",
    "DNSRecursor": "",
    "DNSRecursors": [],
    "Domain": "consul.",
    "LogLevel": "INFO",
    "NodeName": "foobar",
    "ClientAddr": "127.0.0.1",
    "BindAddr": "0.0.0.0",
    "AdvertiseAddr": "10.1.10.12",
    "Ports": {
      "DNS": 8600,
      "HTTP": 8500,
      "RPC": 8400,
      "SerfLan": 8301,
      "SerfWan": 8302,
      "Server": 8300
    },
    "LeaveOnTerm": false,
    "SkipLeaveOnInt": false,
    "StatsiteAddr": "",
    "Protocol": 1,
    "EnableDebug": false,
    "VerifyIncoming": false,
    "VerifyOutgoing": false,
    "CAFile": "",
    "CertFile": "",
    "KeyFile": "",
    "StartJoin": [],
    "UiDir": "",
    "PidFile": "",
    "EnableSyslog": false,
    "RejoinAfterLeave": false
  },
  "Coord": {
    "Adjustment": 0,
    "Error": 1.5,
    "Vec": [0,0,0,0,0,0,0,0]
  },
  "Member": {
    "Name": "foobar",
    "Addr": "10.1.10.12",
    "Port": 8301,
    "Tags": {
      "bootstrap": "1",
      "dc": "dc1",
      "port": "8300",
      "role": "consul",
      "vsn": "1",
      "vsn_max": "1",
      "vsn_min": "1"
    },
    "Status": 1,
    "ProtocolMin": 1,
    "ProtocolMax": 2,
    "ProtocolCur": 2,
    "DelegateMin": 2,
    "DelegateMax": 4,
    "DelegateCur": 4
  }
}
```

### <a name="agent_reload"></a> /v1/agent/reload

Added in Consul 0.7.2, this endpoint is hit with a `PUT` and is used to instruct
the agent to reload its configuration. Any errors encountered during this process
will be returned.

Not all configuration options are reloadable. See the
[Reloadable Configuration](/docs/agent/options.html#reloadable-configuration)
section on the agent options page for details on which options are supported.

The return code is 200 on success.

### <a name="agent_maintenance"></a> /v1/agent/maintenance

The node maintenance endpoint can place the agent into "maintenance mode".
During maintenance mode, the node will be marked as unavailable and will not be
present in DNS or API queries. This API call is idempotent. Maintenance mode is
persistent and will be automatically restored on agent restart.

The `?enable` flag is required. Acceptable values are either `true` (to enter
maintenance mode) or `false` (to resume normal operation).

The `?reason` flag is optional. If provided, its value should be a text string
explaining the reason for placing the node into maintenance mode. This is simply
to aid human operators. If no reason is provided, a default value will be used instead.

The return code is 200 on success.

### <a name="agent_monitor"></a> /v1/agent/monitor

Added in Consul 0.7.2, This endpoint is hit with a GET and will stream logs from the
local agent until the connection is closed.

The `?loglevel` flag is optional.  If provided, its value should be a text string
containing a log level to filter on, such as `info`. If no loglevel is provided,
`info` will be used as a default.

The return code is 200 on success.

### <a name="agent_join"></a> /v1/agent/join/\<address\>

This endpoint is hit with a `GET` and is used to instruct the agent to attempt to
connect to a given address. For agents running in server mode, providing a `?wan=1`
query parameter causes the agent to attempt to join using the WAN pool.

The return code is 200 on success.

### <a name="agent_leave"></a> /v1/agent/leave

Added in Consul 0.7.2, this endpoint is hit with a `PUT` and is used to trigger a
graceful leave and shutdown of the agent. It is used to ensure other nodes see the
agent as "left" instead of "failed". Nodes that leave will not attempt to re-join
the cluster on restarting with a snapshot.

For nodes in server mode, the node is removed from the Raft peer set in a graceful
manner. This is critical, as in certain situations a non-graceful leave can affect
cluster availability.

The return code is 200 on success.

### <a name="agent_force_leave"></a> /v1/agent/force-leave/\<node\>

This endpoint is hit with a `PUT` and is used to instruct the agent to force a node
into the `left` state. If a node fails unexpectedly, then it will be in a `failed`
state. Once in the `failed` state, Consul will attempt to reconnect, and the
services and checks belonging to that node will not be cleaned up. Forcing a node
into the `left` state allows its old entries to be removed.

The endpoint always returns 200.

### <a name="agent_check_register"></a> /v1/agent/check/register

The register endpoint is used to add a new check to the local agent.
There is more documentation on checks [here](/docs/agent/checks.html).
Checks may be of script, HTTP, TCP, or TTL type. The agent is responsible for
managing the status of the check and keeping the Catalog in sync.

The register endpoint expects a JSON request body to be `PUT`. The request
body must look like:

```javascript
{
  "ID": "mem",
  "Name": "Memory utilization",
  "Notes": "Ensure we don't oversubscribe memory",
  "DeregisterCriticalServiceAfter": "90m",
  "Script": "/usr/local/bin/check_mem.py",
  "DockerContainerID": "f972c95ebf0e",
  "Shell": "/bin/bash",
  "HTTP": "http://example.com",
  "TCP": "example.com:22",
  "Interval": "10s",
  "TTL": "15s",
  "TLSSkipVerify": true
}
```

The `Name` field is mandatory, as is one of `Script`, `HTTP`, `TCP` or `TTL`.
`Script`, `TCP` and `HTTP` also require that `Interval` be set.

If an `ID` is not provided, it is set to `Name`. You cannot have duplicate
`ID` entries per agent, so it may be necessary to provide an `ID`.

The `Notes` field is not used internally by Consul and is meant to be human-readable.

In Consul 0.7 and later, checks that are associated with a service may also contain
an optional `DeregisterCriticalServiceAfter` field, which is a timeout in the same Go
time format as `Interval` and `TTL`. If a check is in the critical state for more than
this configured value, then its associated service (and all of its associated checks)
will automatically be deregistered. The minimum timeout is 1 minute, and the
process that reaps critical services runs every 30 seconds, so it may take slightly
longer than the configured timeout to trigger the deregistration. This should
generally be configured with a timeout that's much, much longer than any expected
recoverable outage for the given service.

If a `Script` is provided, the check type is a script, and Consul will
evaluate the script every `Interval` to update the status.

If a `DockerContainerID` is provided, the check is a Docker check, and Consul will
evaluate the script every `Interval` in the given container using the specified
`Shell`. Note that `Shell` is currently only supported for Docker checks.

An `HTTP` check will perform an HTTP `GET` request against the value of
`HTTP` (expected to be a URL) every `Interval`. If the response is any
`2xx` code, the check is `passing`. If the response is `429 Too Many
Requests`, the check is `warning`. Otherwise, the check is `critical`.
HTTP checks also support SSL. By default, a valid SSL certificate is
expected. Certificate verification can be controlled using the
`TLSSkipVerify`.

If `TLSSkipVerify` is set to `true`, certificate verification will be
disabled. By default, certificate verification is enabled.

A `TCP` check will perform an TCP connection attempt against the value of `TCP`
(expected to be an IP or hostname plus port combination) every `Interval`. If the
connection attempt is successful, the check is `passing`. If the connection
attempt is unsuccessful, the check is `critical`. In the case of a hostname
that resolves to both IPv4 and IPv6 addresses, an attempt will be made to both
addresses, and the first successful connection attempt will result in a
successful check.

If a `TTL` type is used, then the TTL update endpoint must be used
periodically to update the state of the check.

The `ServiceID` field can be provided to associate the registered check with an
existing service provided by the agent.

The `Status` field can be provided to specify the initial state of the health
check.

This endpoint supports [ACL tokens](/docs/internals/acl.html). If the query
string includes a `?token=<token-id>`, the registration will use the provided
token to authorize the request. The token is also persisted in the agent's
local configuration to enable periodic
[anti-entropy](/docs/internals/anti-entropy.html) syncs and seamless agent
restarts.

The return code is 200 on success.

### <a name="agent_check_deregister"></a> /v1/agent/check/deregister/\<checkId\>

This endpoint is used to remove a check from the local agent.
The `CheckID` must be passed on the path. The agent will take care
of deregistering the check from the Catalog.

The return code is 200 on success.

### <a name="agent_check_pass"></a> /v1/agent/check/pass/\<checkId\>

This endpoint is used with a check that is of the [TTL type](/docs/agent/checks.html).
When this endpoint is accessed via a `GET`, the status of the check is set to `passing`
and the TTL clock is reset.

The optional `?note=` query parameter can be used to associate a
human-readable message with the status of the check. This will be passed
through to the check's `Output` field in the check endpoints.

The return code is 200 on success.

### <a name="agent_check_warn"></a> /v1/agent/check/warn/\<checkId\>

This endpoint is used with a check that is of the [TTL type](/docs/agent/checks.html).
When this endpoint is accessed via a `GET`, the status of the check is set to `warning`,
and the TTL clock is reset.

The optional `?note=` query parameter can be used to associate a
human-readable message with the status of the check. This will be passed
through to the check's `Output` field in the check endpoints.

The return code is 200 on success.

### <a name="agent_check_fail"></a> /v1/agent/check/fail/\<checkId\>

This endpoint is used with a check that is of the [TTL
type](/docs/agent/checks.html). When this endpoint is accessed via a
`GET`, the status of the check is set to `critical`, and the TTL clock is
reset.

The optional `?note=` query parameter can be used to associate a
human-readable message with the status of the check. This will be passed
through to the check's `Output` field in the check endpoints.

The return code is 200 on success.

### <a name="agent_check_update"></a> /v1/agent/check/update/\<checkId\>

This endpoint is used with a check that is of the [TTL type](/docs/agent/checks.html).
When this endpoint is accessed with a `PUT`, the status and output of the check are
updated and the TTL clock is reset.

This endpoint expects a JSON request body to be put. The request body must look like:

```javascript
{
  "Status": "passing",
  "Output": "curl reported a failure:\n\n..."
}
```

The `Status` field is mandatory, and must be set to `passing`,
`warning`, or `critical`.

`Output` is an optional field that will associate a human-readable
message with the status of the check, such as the output of the checking
script or process. This will be truncated if it exceeds 4KB in size.
This will be passed through to the check's `Output` field in the check
endpoints.

The return code is 200 on success.

### <a name="agent_service_register"></a> /v1/agent/service/register

The register endpoint is used to add a new service, with an optional
health check, to the local agent. There is more documentation on
services [here](/docs/agent/services.html). The agent is responsible
for managing the status of its local services, and for sending updates
about its local services to the servers to keep the global Catalog in
sync.

The register endpoint expects a JSON request body to be `PUT`. The request
body must look like:

```javascript
{
  "ID": "redis1",
  "Name": "redis",
  "Tags": [
    "primary",
    "v1"
  ],
  "Address": "127.0.0.1",
  "Port": 8000,
  "EnableTagOverride": false,
  "Check": {
    "DeregisterCriticalServiceAfter": "90m",
    "Script": "/usr/local/bin/check_redis.py",
    "HTTP": "http://localhost:5000/health",
    "Interval": "10s",
    "TTL": "15s"
  }
}
```

The `Name` field is mandatory. If an `ID` is not provided, it is set to
`Name`. You cannot have duplicate `ID` entries per agent, so it may be
necessary to provide an ID in the case of a collision.

`Tags`, `Address`, `Port`, `Check` and `EnableTagOverride` are optional.

If `Address` is not provided or left empty, then the agent's address will be used
as the address for the service during DNS queries. When querying for services using
HTTP endpoints such as [service health](/docs/agent/http/health.html#health_service)
or [service catalog](/docs/agent/http/catalog.html#catalog_service) and encountering
an empty `Address` field for a service, use the `Address` field of the agent node
associated with that instance of the service, which is returned alongside the service
information.

If `Check` is provided, only one of `Script`, `HTTP`, `TCP` or `TTL`
should be specified. `Script` and `HTTP` also require `Interval`. The
created check will be named `service:<ServiceId>`.

In Consul 0.7 and later, checks that are associated with a service may
also contain an optional `DeregisterCriticalServiceAfter` field, which
is a timeout in the same [Go time format](https://golang.org/pkg/time/)
as `Interval` and `TTL`. If a check is in the critical state for more
than this configured value, then its associated service (and all of its
associated checks) will automatically be deregistered. The minimum
timeout is 1 minute, and the process that reaps critical services runs
every 30 seconds, so it may take slightly longer than the configured
timeout to trigger the deregistration. This should generally be
configured with a timeout that is much, much longer than any expected
recoverable outage for the given service.

There is more information about checks [here](/docs/agent/checks.html).

The `EnableTagOverride` option can be specified to disable the
anti-entropy feature for this service's tags. If `EnableTagOverride` is
set to `true` then external agents can update this service in the
[catalog](/docs/agent/http/catalog.html) and modify the tags. Subsequent
local sync operations by this agent will ignore the updated tags. For
instance, if an external agent modified both the tags and the port for
this service and `EnableTagOverride` was set to `true` then after the
next sync cycle the service's port would revert to the original value
but the tags would maintain the updated value. As a counter example, if
an external agent modified both the tags and port for this service and
`EnableTagOverride` was set to `false` then after the next sync cycle
the service's port _and_ the tags would revert to the original value and
all modifications would be lost.

It's important to note that this applies only to the locally registered
service. If you have multiple nodes all registering the same service
their `EnableTagOverride` configuration and all other service
configuration items are independent of one another. Updating the tags
for the service registered on one node is independent of the same
service (by name) registered on another node. If `EnableTagOverride` is
not specified the default value is `false`. See [anti-entropy
syncs](/docs/internals/anti-entropy.html) for more info.

This endpoint supports [ACL tokens](/docs/internals/acl.html). If the query
string includes a `?token=<token-id>`, the registration will use the provided
token to authorize the request. The token is also persisted in the agent's
local configuration to enable periodic
[anti-entropy](/docs/internals/anti-entropy.html) syncs and seamless agent
restarts.

The return code is 200 on success.

### <a name="agent_service_deregister"></a> /v1/agent/service/deregister/\<serviceId\>

The deregister endpoint is used to remove a service from the local agent.
The `ServiceID` must be passed after the slash. The agent will take care
of deregistering the service with the Catalog. If there is an associated
check, that is also deregistered.

The return code is 200 on success.

### <a name="agent_service_maintenance"></a> /v1/agent/service/maintenance/\<serviceId\>

The service maintenance endpoint allows placing a given service into
"maintenance mode". During maintenance mode, the service will be marked as
unavailable and will not be present in DNS or API queries. This API call is
idempotent. Maintenance mode is persistent and will be automatically restored
on agent restart. The maintenance endpoint expects a `PUT` request.

The `?enable` flag is required. Acceptable values are either `true` (to enter
maintenance mode) or `false` (to resume normal operation).

The `?reason` flag is optional. If provided, its value should be a text string
explaining the reason for placing the service into maintenance mode. This is simply
to aid human operators. If no reason is provided, a default value will be used instead.

The return code is 200 on success.
