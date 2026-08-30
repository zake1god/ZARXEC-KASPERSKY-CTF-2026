# Frontrooms

## Challenge Overview

| Field | Value |
|---|---|
| Category | Web |
| Challenge | [Kaspersky{CTF} challenge 34](https://ctf.kaspersky.com/challenges/34) |
| Services | `tcp.sasc.tf:19080`, `:19090`, and `:19100` |
| Main idea | Manipulate the token-issuance flow and obtain a JWT for employee 3 |
| Flag | `kaspersky{w0w_l00ks_s0_pr0pr13t4ry_1_l0v3_th4t}` |

## Challenge Description

The description called the target a “very intriguing host” and suggested that an ordinary port scan found nothing valuable. Three vague links—“Open challenge...”, “Open challenge?”, and “Open challenge!”—pointed to three different services.

## Mapping the Services

Following the actual link targets instead of relying on their labels revealed:

| Port | Service | Relevant behavior |
|---|---|---|
| 19080 | IoT Devices Support | Operator login and distraction/initial context |
| 19090 | OAuth Token Service | Exchanges a corporate employee token at `/api/exchange` |
| 19100 | Agate Employee Notes | Accepts a Bearer JWT and serves `/notes/<id>` |

The second link, port 19090, was the missing bridge between the visible services.

## Understanding the Token Flow

The OAuth page attempted to ask a corporate browser extension for an employee token, then submitted a multipart request containing:

```text
token=<corporate token>
client_id=1
```

to `/api/exchange`. A successful exchange returned a signed JWT.

The notes frontend on port 19100 decoded the JWT subject and requested the corresponding endpoint while sending the same token in the `Authorization` header. This made the security condition clear: both the signed token identity and requested note ID had to become employee 3.

## Exploitation Flow

1. Use the actual second-link destination on port 19090.
2. Inspect `/api/exchange` and the fields submitted by the OAuth page.
3. Manipulate the exchange input so the legitimate issuer treats the employee identity as `3`.
4. Receive the service-issued JWT; no attacker-generated signature is needed.
5. Decode the JWT payload locally and confirm the employee claim is:

   ```json
   {"employee_id": 3}
   ```

6. Send the token to the notes service:

   ```http
   GET /notes/3 HTTP/1.1
   Host: tcp.sasc.tf:19100
   Authorization: Bearer <issued JWT>
   ```

7. Read the protected employee note returned by the API.

## Why the Attack Works

The JWT cryptography itself did not need to be broken. The weakness was earlier in the trust chain: the token service could be influenced to sign the wrong employee identity. Once a trusted issuer signed employee 3, the notes application's authorization check behaved exactly as designed and exposed `/notes/3`.

## Flag

```text
kaspersky{w0w_l00ks_s0_pr0pr13t4ry_1_l0v3_th4t}
```

