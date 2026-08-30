# lumberjack.ai

## Challenge Overview

| Field | Value |
|---|---|
| Category | AI |
| Challenge | [Kaspersky{CTF} challenge 15](https://ctf.kaspersky.com/challenges/15) |
| Target | ONNX model-upload service |
| Main idea | Abuse a negative tree feature index as an out-of-bounds read primitive |
| Flag | `kaspersky{47bd029e-4144-48d5-adc3-8fe034fb7ae2}` |

## Challenge Description

The service accepted an uploaded ONNX forest model, ran it against a fixed input containing 40 features, and returned the predicted class and probability. The flag was not part of the model input, but it was copied into process memory immediately before the feature buffer.

This layout turned a model-validation mistake into a memory-disclosure challenge.

## Initial Analysis

An ONNX `TreeEnsembleClassifier` stores the feature used by every node in `nodes_featureids`. Normally these indices should refer to one of the 40 input features. The runtime used by the service, however, also accepted negative indices.

With the flag buffer placed directly before feature zero, a negative feature ID read four bytes from an address before the legitimate tensor:

```text
[ flag buffer, padded to 48 bytes ][ 40 model features ]
                                   ^ feature 0
```

The useful starting index was `-20`. Each selected feature value therefore exposed one little-endian 32-bit word from the preceding memory. Incrementing the negative index moved through the padded flag buffer one word at a time.

## Building an Observable Read Primitive

A single comparison would reveal only one bit per upload. To reduce the number of requests, the malicious tree was built as a balanced interval classifier:

1. Treat the leaked four bytes as the raw bit pattern of a 32-bit float.
2. Split the current candidate range into as many as 400 intervals.
3. Construct a tree whose leaves identify the interval containing the secret value.
4. Encode the leaf number using two observable fields from the response:
   - 8 possible output classes;
   - 50 distinguishable probability values.
5. Decode `class x 50 + probability_bucket` to recover one of 400 intervals.
6. Repeat with the smaller range until only one 32-bit value remains.

This is effectively a high-radix binary search. The model never prints memory directly; its control flow converts a secret comparison result into an output class and probability.

## Extraction Flow

For each flag word:

1. Generate an ONNX forest using the current negative feature ID.
2. Upload the model and record the returned class and probability.
3. Convert that pair to the chosen interval number.
4. Narrow the candidate IEEE-754 bit range.
5. Repeat until the exact 32-bit word is known.
6. Pack the result as little-endian bytes.
7. Move to the next negative feature index.

Twelve words covered the complete 48-byte buffer. Extraction stopped at the zero terminator, and concatenating the recovered bytes produced the flag string.

## Why the Attack Works

The service trusted structural ONNX metadata to address the input tensor. Because negative `nodes_featureids` values were not rejected, model evaluation could read outside the tensor. The classifier's normal output then acted as a covert channel for those out-of-bounds values.

## Flag

```text
kaspersky{47bd029e-4144-48d5-adc3-8fe034fb7ae2}
```
