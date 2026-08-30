# Canary

## Challenge Overview

| Field | Value |
|---|---|
| Category | Web |
| Challenge | [Kaspersky{CTF} challenge 30](https://ctf.kaspersky.com/challenges/30) |
| Target | URL-scanning browser bot |
| Main idea | DNS rebind the bot to IMDSv2, recover its IAM role, and read a constrained S3 object |
| Flag | `kaspersky{5688e9f664a7fa9bce6bbe8414ee6423}` |

## Challenge Description

The site accepted a URL and sent a browser bot to inspect it as a security add-on for mail gateways. The attack surface was therefore the bot's browser and its internal network position.

## Initial Dead End

Trying to steal an autofilled credential returned only a decoy. The more useful weakness was in URL validation: the service validated a public hostname before navigation but did not bind that hostname to one IP for the bot's full visit.

## DNS Rebinding to IMDS

1. Serve an attacker-controlled hostname that initially resolves to a public server.
2. Pass the application's URL checks and load the attacker page in the bot.
3. Change the DNS answer to `169.254.169.254` after the first resolution.
4. Use staggered navigation attempts so a later lookup reaches AWS Instance Metadata Service.

Chrome's Private Network Access behavior blocked ordinary fetches and normal subframes. Credentialless iframes used a separate network-isolation state and provided the reliable path. Multiple staggered frames handled the race between DNS cache expiration and navigation.

IMDSv2 required two steps:

1. Send the metadata-token `PUT` request with the TTL header.
2. Reuse the returned token for the user-data and IAM credential requests.

## AWS Credential Flow

The metadata responses exposed instance user-data and temporary IAM role credentials. User-data named the relevant artifact bucket.

The role could not list the bucket without a condition. Its policy allowed listing only when the requested prefix matched `reports*`, so the correct enumeration request used prefix `reports`.

That listing revealed:

```text
reports/golden/reported-message.eml
```

I fetched that exact object with the temporary role credentials. The small regression-test email contained an internal recovery reference, which was the actual flag.

## Root Cause

The chain combined DNS rebinding, a browser-network-policy gap for credentialless frames, an instance role reachable through IMDS, and an S3 policy that restricted listing but still disclosed the target under an allowed prefix.

## Flag

```text
kaspersky{5688e9f664a7fa9bce6bbe8414ee6423}
```

