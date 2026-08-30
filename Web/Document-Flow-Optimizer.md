# Document Flow Optimizer (html2pdf)

## Challenge Overview

| Field | Value |
|---|---|
| Category | Web |
| Challenge | [Kaspersky{CTF} challenge 23](https://ctf.kaspersky.com/challenges/23) |
| Source | `server_301c81a7bdfc27aa92ec0a79635a1ff8_src` |
| Main idea | Traverse a shared renderer cache and overwrite a periodic healthcheck with a PNG/shell polyglot |
| Flag | `kaspersky{c8e36ca5-ab9e-4b1c-84ed-5c2e7eeec542}` |

## Challenge Description

The service rendered a supplied URL into a PDF and offered only twenty free generations. The Flask wrapper delegated rendering to a custom `html2pdf` binary based on PlutoBook 0.19.0.

## Establishing the Target

The flag was root-only, but the container included a SUID helper that printed it when invoked as:

```sh
/readflag please
```

The renderer process could not read the flag directly. The useful target was therefore a file later executed by a more privileged or trusted process: `/app/healthcheck.sh`, periodically run by Docker.

## Cache Write Primitive

The renderer stored shared images under `/app/work/.cache`. Its cache key/path construction had two relevant flaws:

- different iframe URLs could collide on the same cache filename;
- encoded traversal components followed by a null byte could redirect the final PNG write outside the cache directory.

Together these behaviors provided a controlled file overwrite, but the renderer always wrote PNG bytes.

## Building the PNG/Shell Polyglot

The replacement healthcheck had to remain a valid renderer-produced PNG while also containing shell syntax at executable offsets. The embedded command was equivalent to:

```sh
cd app/static
/readflag please > x
```

The renderer recompressed image data, so simply inserting text into a PNG was not enough. The forging helper adjusted selected pixel tokens while keeping the DEFLATE Huffman histogram stable. This preserved the command bytes after recompression. A small image kept the render below the 100 MB limit.

## Exploitation Flow

1. Host a page containing an iframe whose encoded URL maps outside the shared cache.
2. Make the destination resolve to `/app/healthcheck.sh`.
3. Serve the crafted image input and request one PDF generation.
4. The renderer caches/recompresses it and overwrites the healthcheck script.
5. Wait for Docker's next periodic healthcheck.
6. The shell reaches the embedded command and runs the SUID helper.
7. Fetch `/static/x`, which now contains the flag.

## Helpers

- [`exploit_server.py`](../../Compressed/server_301c81a7bdfc27aa92ec0a79635a1ff8_src/lab/exploit_server.py)
- [`forge_row.py`](../../Compressed/server_301c81a7bdfc27aa92ec0a79635a1ff8_src/lab/forge_row.py)

## Flag

```text
kaspersky{c8e36ca5-ab9e-4b1c-84ed-5c2e7eeec542}
```

