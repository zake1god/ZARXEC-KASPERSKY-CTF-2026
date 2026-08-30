# Cloud Storage

## Challenge Overview

| Field | Value |
|---|---|
| Category | Web |
| Challenge | [Kaspersky{CTF} challenge 22](https://ctf.kaspersky.com/challenges/22) |
| Protocol | gRPC |
| Main idea | Accumulate a shared default billing object, then inject ZODB configuration |
| Flag | `kaspersky{14192d7e-c5e9-4fb6-8af9-65179687db7a}` |

## Challenge Description

The source described a cloud-storage investment service. New accounts received small random grants, while creating storage cost 7331 credits—far beyond one user's balance.

## Bug 1: Persistent Mutable Default Argument

The billing layer defined:

```python
def create(self, user_id, billing=copy.deepcopy(default_billing)):
```

Python evaluates default arguments once when the function is defined. The copied `Billing` object was therefore shared by every call that omitted the argument.

`create()` changed this SQLAlchemy object's owner and UUID in place but did not reset its balance. The `Grant` RPC made the bug exploitable because a nonexistent billing ID caused that same default object to be created and credited.

## Credit Accumulation Flow

1. Register a fresh user and log in.
2. Call `Grant` with a new nonexistent billing ID.
3. The service moves the shared billing object to the current user and adds 10–50 credits.
4. Register another user and repeat with another missing ID.
5. Track the returned balance; it survives ownership changes and continues increasing.
6. Once the shared balance exceeds 7331, remain logged in as its latest owner.
7. Call `List` to recover the current spendable billing UUID.

## Bug 2: ZODB Configuration Injection

The storage name was inserted into:

```text
<filestorage>
    path storages/{storage_id}/{name}_storage
</filestorage>
```

Validation normalized the input as a filesystem path but did not reject newlines. A name could terminate the `path` directive and append a new ZODB directive:

```text
retirement-<random-tag>
packer os:(_ for _ in ()).throw(Exception(popen('/readflag please').read()))
#
```

## Code-Execution Flow

1. Buy/create storage using the accumulated billing UUID.
2. Supply the multiline storage name above.
3. The service writes the injected text to `storage.conf`.
4. Call the `Keys` RPC, causing `ZODB.config.storageFromURL()` to parse the file.
5. The injected `packer` expression runs `/readflag please`.
6. The expression raises an exception containing stdout.
7. Read the flag from the gRPC error details.

## Verification and Solver

The solver checks the balance after every grant and only attempts storage creation after the threshold is reached.

Solver: [`solve_cloud_storage.py`](../../Compressed/cloud-storage_analysis/solve_cloud_storage.py)

## Flag

```text
kaspersky{14192d7e-c5e9-4fb6-8af9-65179687db7a}
```

