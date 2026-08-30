# Garden

## Challenge Overview

| Field | Value |
|---|---|
| Category | Web |
| Challenge | [Kaspersky{CTF} challenge 1](https://ctf.kaspersky.com/challenges/1) |
| Main idea | Error-based SQLite injection through a syntactically valid Shogi move |
| Flag | `kaspersky{0ld_s4mur4i_w0u1d_b3_pr0u6}` |

## Challenge Description

Garden was a web Shogi game backed by SQLite. The admin's first stored move contained the flag, but normal users could only submit moves for their own games.

## Attack Surface

The game parser validated a move before the database layer inserted it. The two components did not agree on where the move ended: the parser accepted a valid prefix, while the SQL builder consumed the entire string.

I first played a short sequence of legal moves so that a promoted-pawn/drop-style move beginning with `P*1e` was valid in the current game state. This was necessary; injecting into a move rejected by the game engine never reached SQLite.

## Building the Error Oracle

The target value was selected from `game_moves`:

```sql
SELECT move
FROM game_moves
WHERE user_id = 'admin'
ORDER BY id ASC
LIMIT 1
```

SQLite's `json_extract()` includes an invalid JSON path in its error message. The stored flag is not a valid JSON path, so using it as that argument turns the database error into an exfiltration channel.

A shortened payload was:

```sql
P*1e' || json_extract('{}',
  (SELECT move FROM game_moves
   WHERE user_id='admin' ORDER BY id ASC LIMIT 1)
), datetime('now')) --
```

## Exploitation Flow

1. Register/login and create a game.
2. Play the prerequisite legal sequence.
3. Submit a move whose prefix `P*1e` satisfies the Shogi parser.
4. Let the unconsumed suffix break out of the SQL string.
5. Select the admin's oldest stored move.
6. Pass it to `json_extract` as a path.
7. Read the SQLite exception reflected in the HTTP response.
8. Retry if the shared instance returns `database is locked`.

## Root Cause

Validation was performed on a different interpretation of the input than the one used by the SQL query. A valid application-level prefix did not make the full string safe for string-concatenated SQL.

## Flag

```text
kaspersky{0ld_s4mur4i_w0u1d_b3_pr0u6}
```

