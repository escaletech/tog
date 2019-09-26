# Tog

Tog (short for toggle) is a framework for clients and servers to converse about feature flags over Redis.

1. [Concepts](#concepts)
    1. [Flags](#flags)
    2. [Experiments](#experiments)
    3. [Sessions](#sessions)
2. [Client operations](#client-operations)
    1. [List flags](#list-flags)
    2. [Get a flag](#get-a-flag)
    3. [List a session's flags](#list-a-sessions-flags)
    4. [Create an experiment](#create-an-experiment)
3. [Ecosystem](#ecosystem)

## Concepts

### Flags

A flag and its current value.

**Type**: value
**Key**: `flag:{namespace}:{flag_name}`
**Value**: `1` (on) or `0` (off)

The value MAY be appended by a semicolon followed by a free-text description. Example: `1;Sets the call-to-action button color to blue`.


### Experiments

An ongoing experiment.

**Type**: hash
**Key**: `exp:{namespace}:{experiment_name}`
**Fields**: `@weight=0.3`, `@disabled`, `foo=1`, `bar=0`, `@desc=Some free-text description`

A tag field `@weight` states the weight of this experiment when drawing values for a session.

The tag field `@disabled` disables an experiment, thus not considering it for new sessions.

### Sessions

A hash of flag values for a session.

**Type**: hash
**Key**: `session:{namespace}:{session_id}`
**Fields**: key/value, where key is each flag's name


## Client Operations

A client implementation MAY adopt a memory cache for a namespace's flag list. If it chooses do do so, it MUST subscribe to changes and clear its cache either when notified or reconnected.

### List flags

Time: `1`

1. `KEYS flag:{namespace}:*`
2. `MGET {key_1} {key_2} {key_n}`

### Get a flag

Time: `0`

1. Try to find a previously listed flag from cache
2. If not found or not connected, return a default value

### List a session's flags

Time: `n_experiments`

1. `HGETALL session:{namespace}:{session_id}`; if found, return it
2. `KEYS exp:{namespace}:*`
3. For each experiment, `HGETALL exp:{namespace}:{experiment_name}`
4. Discard experiments tagged with `@disabled`
5. Randomly pick an experiment, taking `@weight` into consideration
6. `HMSET session:{namespace}:{session_id} {...field-values}` with the randomly picked values
7. `SET exp_sess:{namespace}:{experiment_name}:{session_id}` to bind a session to the experiment

### Create an experiment

Time: `1`

1. `HMSET exp:{namespace}:{experiment_name} {...field-values}`


## Ecosystem

* [Management server](https://github.com/escaletech/tog-management-server): server application that provides management of flags and experiments. Best used with Tog CLI.
* [Session server](https://github.com/escaletech/tog-session-server): server application that provides an endpoint for fetching sessions
* [CLI](https://github.com/escaletech/tog-cli): command-line tool that interacts with the management server to update flags and experiments
