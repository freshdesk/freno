# freno

Cooperative, highly available throttler service: clients can use `freno` to throttle writes to a resource.

Current implementation can throttle writes to (multiple) MySQL clusters, based on replication status for those clusters. `freno` will throttle cooperative clients when replication lag exceeds a pre-defined threshold.

`freno` dynamically adapts to changes in server inventory; it can further be controlled by the user to force throttling of certain apps.

`freno` is highly available and uses `raft` consensus protocol to decide leadership and to pass user events between member nodes.


### Cooperative

`freno` collects data from backend stores (at this time MySQL only) and has the logic to answer the question "may I write to the backend store?"

Clients (application, scripts, jobs) are expected to consult with `freno`. `freno` is not a proxy between the client and the backend store. It merely observes the store and states "you're good to write" or "you should stop writing". Clients are expected to consult with `freno` and respect its recommendation.

### Per store

### Per app

### MySQL


### Use cases

`freno` is useful for bulk operations: massive loading/archiving tasks, schema migrations, mass updates. Such operations typically walk through thousands to millions of rows and may cause undesired effects such as MySQL replication lags. By breaking these tasks to small subtasks (e.g. `100` rows at a time), and by consulting `freno` before applying each such subtask, we are able to achieve the same result without ill effect to the database and to the application that uses it.

`freno` is not suitable for OLTP queries.

### HTTP

`freno` serves requests via `HTTP`. The most important request is the `check` request: "May this app write to this store?". `freno` appreciates `HEAD` requests (`GET` are also accepted, with more overhead) and responds with status codes:

- `200` (OK): Application may write to data store
- `417` (Expectation Failed): Requesting application is explicitly forbidden to write.
- `429` (Too Many Requests): Do not write. A normal state indicating the store's state does not meet expected threshold.
- `500` (Internal Server Error): Internal error. Do not write.

Read more on [HTTP requests & responses](doc/http.md)

### Raft

Read more on `raft` and [High Availability](doc/high-abailability.md)

### HAProxy

Read more on [High Availability](doc/high-abailability.md)

### Configuration

### What's in a name?

"Freno" is Spanish for "brake", as in _car brake_. Basically we just wanted to call it "throttler" or "throttled" but both these names are in use by multiple other repositories and we went looking for something else. When we looked up the word "freno" in a dictionary, we found the following sentence:

> Echa el freno, magdaleno!

This reminded us of the 80's and that was it.
