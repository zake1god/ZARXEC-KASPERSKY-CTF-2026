# Kube Adventure

## Challenge Overview

| Field | Value |
|---|---|
| Category | Web |
| Challenge | [Kaspersky{CTF} challenge 8](https://ctf.kaspersky.com/challenges/8) |
| Environment | Flagger service inside Kubernetes |
| Main idea | Chain command execution, registry leakage, Kubernetes identity escalation, and Redis credential interception |
| Flag | `kaspersky{d5700008-7436-441b-8f83-33602ce32b6c}` |

## Challenge Description

The public application exposed a Flagger load-testing interface. Command execution in that pod was only the first foothold; the flag lived behind several Kubernetes trust boundaries.

## Stage 1: Flagger Command Execution

Flagger webhooks supported a command form with `type: bash`. Enabling `returnCmdOutput` made the service return stdout to the caller.

The resulting shell ran as the unprivileged `app` user. Initial checks established useful negative facts:

- the flag was not present in the pod filesystem;
- no normal service-account token was mounted;
- anonymous Kubernetes API requests were denied.

This prevented wasting time on a direct `kubectl` or local-secret path.

## Stage 2: Recovering Node Credentials

1. Enumerate reachable internal services from the pod.
2. Find the unauthenticated/internal container registry.
3. Request its catalog and identify challenge-specific images, including `node-recovery`.
4. Download the image manifest and layers as data.
5. Search the extracted filesystem and recover a valid Kubernetes bootstrap token.
6. Create a short-lived bootstrap CSR for the existing node identity.
7. Retrieve the signed certificate and use it as node credentials.

## Stage 3: Moving Through Kubernetes Authorization

The node identity could not read arbitrary secrets or execute inside arbitrary pods. However, the Node Authorizer permitted a bound token request for the `kube-state-metrics` service account associated with a pod on that node.

Using that service-account token exposed cluster inventory. The important discovery was a `rollout-observer` that repeatedly authenticated to Redis through a fixed Service IP that currently had no legitimate Service behind it.

## Stage 4: Intercepting Redis Authentication

1. Use the limited publisher permission to create a temporary Service claiming the expected free VIP.
2. Point that Service at a transparent TCP proxy running in the already-controlled load-tester pod.
3. Forward all traffic to the real Redis server so the observer continues to work.
4. Log the Redis protocol long enough to capture the observer's `AUTH` password.
5. Remove or stop relying on the interception Service once the credential is recovered.
6. Authenticate directly to Redis with the captured read-only credential.
7. Read:

   ```text
   rollout:production:flag
   ```

## Why the Chain Works

No single identity was cluster-admin. The solve combined narrowly scoped capabilities: Flagger command execution exposed the registry; the leaked bootstrap token became a node identity; node authorization enabled a bound service-account token; and that account revealed enough topology to abuse an unclaimed fixed Service IP.

## Flag

```text
kaspersky{d5700008-7436-441b-8f83-33602ce32b6c}
```

