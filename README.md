# Tog Spec (v0.3)

Tog (short for toggle) is a framework for clients and servers to converse about feature flags over Redis.

- [Tog Spec (v0.3)](#tog-spec-v03)
  - [Concepts](#concepts)
    - [Flags](#flags)
    - [Rollout strategies](#rollout-strategies)
  - [Client Operations](#client-operations)
    - [List flags](#list-flags)
    - [Get a flag](#get-a-flag)
    - [Save a flag](#save-a-flag)
    - [Delete a flag](#delete-a-flag)
    - [List a session's flags](#list-a-sessions-flags)
  - [Ecosystem](#ecosystem)

## Concepts

### Flags

A flag and its current value.

**Type**: Hash Map
**Key**: `tog3:flags:{namespace}`
**Field**: `{name}`
**Value**:

```json
{
  "description": "Sets the call-to-action button color to blue",
  "timestamp": 1590748359,
  "rollout": [
    {
      "percentage": 30,
      "value": true
    },
    {
      "traits": ["early_adopter"],
      "value": true
    },
    {
      "value": false
    }
  ]
}
```

### Rollout strategies

If multiple strategies are defined for one rollout option, they will be applied with the AND operator. If no rollout rule is matched, then the fallback value for the flag is `false`.

- `percentage`: will randomly assign the value to X percent of the sessions. Example:
  - `{ "percentage": 30 }`
- `traits`: will assign the value when the session's traits match all of the rollout's traits. Example:
  - `{ "traits": ["early_adopter"] }`

Random assignment of percentages MUST follow this criteria:

- Key = take the input session ID and concatenate it with the flag's timestamp
- Hash = use the previous string as key for computing a Murmur3 (32b) hash
- Mod = Take the modulo of the division of the hash by 100
- Consider the rollout expression if Mod < percentage

## Client Operations

A client implementation MAY adopt a memory cache for a namespace's flag list, in which case it must subscribe to `tog3:namespace-changed` to clear the cache when a change happens.

### List flags

1. `HGETALL tog3:flags:{namespace}`

### Get a flag

1. `HGET tog3:flags:{namespace} {name}`

### Save a flag

1. `HSET tog3:flags:{namespace} {name} {flag-json}`
2. `PUBLISH tog3:namespace-changed {namespace}`

### Delete a flag

1. `HDEL tog3:flags:{namespace} {name}`
2. `PUBLISH tog3:namespace-changed {namespace}`

### List a session's flags

1. List flags
2. For each flag, apply its rollout rules to figure out its state

## Ecosystem

- Clients:
  - Node.js: [`tog-node`](https://github.com/escaletech/tog-node)
- [Management server](https://github.com/escaletech/tog-management-server): server application that provides management of flags and experiments. Best used with Tog CLI.
- [Session server](https://github.com/escaletech/tog-session-server): server application that provides an endpoint for fetching sessions
- [CLI](https://github.com/escaletech/tog-cli): command-line tool that interacts with the management server to update flags and experiments
