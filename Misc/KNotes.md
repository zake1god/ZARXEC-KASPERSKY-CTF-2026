# KNotes

## Challenge Overview

| Field | Value |
|---|---|
| Category | Misc |
| Challenge | [Kaspersky{CTF} challenge 9](https://ctf.kaspersky.com/challenges/9) |
| Main idea | Execute pull-request code in a privileged `pull_request_target` workflow |
| Flag | `kaspersky{c701669e-1fe7-4a52-8c95-edea9d788277}` |

## Challenge Description

The only textual clue was “Open Source is great! Or is it?” The notes site linked to a public Gitea repository, while the flag was stored in a private admin note.

## Source Review

The Flask application used an `ADMIN_TOKEN` to unlock `/admin`. That route displayed all notes, including the private flag note. The token was not committed to the repository, but it was available as a CI secret.

The repository's workflow ran on `pull_request_target`. This event executes in the trusted base-repository context and can access its secrets. That is only safe if the workflow never executes files controlled by the pull request.

Here the job checked out pull-request content and later launched command-line tools while `ADMIN_TOKEN` remained in the environment.

## Exploitation Flow

1. Fork the repository and create a minimal pull request.
2. Add an executable wrapper named `sed` in a path searched before the system binary.
3. Allow the privileged workflow to check out and run the pull-request version.
4. When a later workflow step invokes `sed`, the wrapper executes with `ADMIN_TOKEN` in its environment.
5. Avoid secret masking. Plain output and ordinary Base64 are redacted, so print each byte as a separate hexadecimal item.
6. Read the CI log and concatenate the hex bytes locally.
7. Submit the recovered token to `/admin`.
8. Open note `#1 - Flag`.

## Root Cause

The issue was not `pull_request_target` by itself. The dangerous combination was:

```text
trusted event context + repository secrets + checkout of untrusted PR code + execution
```

The job crossed the trust boundary after secrets had already been granted.

## Verification

The reconstructed token successfully unlocked the real admin page. The flag came from the private note rather than from a value guessed or exposed directly in CI.

## Flag

```text
kaspersky{c701669e-1fe7-4a52-8c95-edea9d788277}
```

