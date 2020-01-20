# Tog

Tog (short for toggle) is a framework for clients and servers to converse about feature flags over Redis.

- [Tog](#tog)
  - [Concepts](#concepts)
    - [Flags](#flags)
    - [Sessions](#sessions)
    - [Rollout strategies](#rollout-strategies)
  - [Client Operations](#client-operations)
    - [List flags](#list-flags)
    - [Get a flag](#get-a-flag)
    - [List a session's flags](#list-a-sessions-flags)
  - [Ecosystem](#ecosystem)

## Concepts

### Flags

A flag and its current value.

**Type**: value
**Key**: `tog2:flag:{namespace}:{flag_name}`
**Value**:
```json
{
  "description": "Sets the call-to-action button color to blue",
  "rollout": [
    {
      "percentage": 30,
      "value": true
    },
    {
      "value": false
    }
  ]
}
```

### Sessions

A hash of flag values for a session.

**Type**: hash
**Key**: `tog2:session:{namespace}:{session_id}`
**Value**: 
```json
{
    "<flag-one>": true,
    "<flag-two>": false
}
```

### Rollout strategies

If multiple strategies are defined for one rollout option, they will be applied with the AND operator. If no rollout rule is matched, then the fallback value for the flag is `false`.

* `percentage`: will randomly assign the value to X percent of the sessions. Example:
  * `{ "percentage": 30 }`
* `tags`: will assign the value when the session's tags match all of the rollout tags. Example:
  * `{ "match": { "domain": "escale.com.br" } }`


## Client Operations

A client implementation MAY adopt a memory cache for a namespace's flag list. If it chooses to do so, it MUST subscribe to changes and clear its cache either when notified or reconnected.

### List flags

1. `KEYS tog2:flag:{namespace}:*`
2. `MGET {key_1} {key_2} {key_n}`

### Get a flag

1. Try to find a previously listed flag from cache
2. If not found or not connected, return a default value

### List a session's flags

1. `GET tog2:session:{namespace}:{session_id}`; if found, return it
2. List flags
3. For each flag, apply its rollout rules to figure out its state
4. `SET tog2:session:{namespace}:{session_id} {session-json}` with the picked values


## Ecosystem

* Clients:
  * Node.js: [`tog-node`](https://github.com/escaletech/tog-node)
* [Management server](https://github.com/escaletech/tog-management-server): server application that provides management of flags and experiments. Best used with Tog CLI.
* [Session server](https://github.com/escaletech/tog-session-server): server application that provides an endpoint for fetching sessions
* [CLI](https://github.com/escaletech/tog-cli): command-line tool that interacts with the management server to update flags and experiments
