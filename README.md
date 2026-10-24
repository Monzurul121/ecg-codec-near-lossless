# Near-Lossless ECG Compression

A low-complexity, near-lossless codec for single-lead ECG signals. It combines 11-bit quantization, second-order delta coding and buffer-based run-length coding to shrink the signal while keeping the waveform essentially unchanged.

> **Status: preview.** This repository documents the method and its interface. The implementation will be released later.

## Overview

Wearable and long-term ECG monitors record for hours or days. Storing or sending every sample at full precision quickly fills memory and uses up bandwidth. Lossless and near-lossless codecs reduce this cost without altering the diagnostic waveform.

This codec follows a real-time lossless scheme designed for ECG and PPG signals. It uses only integer arithmetic and simple operations, which keeps it fast and suitable for resource-limited devices.

It is called *near-lossless* because its first step, 11-bit quantization, introduces a small error relative to the floating-point input. Every later step is exactly reversible. The codec measures this error rather than assuming it is zero.

## How it works

1. **11-bit quantization.** The record's amplitude range is mapped onto an 11-bit integer grid, with rounding. This is the only lossy step: each sample moves by at most half a grid step.
2. **Second-order delta coding.** The codec keeps the first sample and the first difference. It then replaces every other sample with its second-order difference, which is the change in slope. Because an ECG changes smoothly most of the time, these values are small and many are exactly zero.
3. **Buffer-based coding.** The second-order differences are routed into three buffers:
   - a main buffer of short, fixed-width codes for the frequent small values;
   - a run-length buffer that stores runs of consecutive zeros compactly;
   - an escape buffer for the rare large values, such as those at steep QRS complexes, stored at full width.

   A reserved code in the main buffer tells the decoder when to read from the other two buffers.
4. **Automatic bit-width selection.** For each record, the codec tries several code widths for the main buffer and keeps the one that gives the smallest total size.
5. **Packing.** The three buffers and a small header (quantization range, first values and code widths) are packed into a single bitstream.
6. **Decoding.** The decoder unpacks the buffers, rebuilds the second-order differences, integrates them twice and maps the integers back to amplitudes.

## Key properties

- **Near-lossless:** the reconstruction differs from the input only by the 11-bit quantization.
- **Low complexity:** integer arithmetic and single-pass coding, with no transforms and no training.
- **Self-contained payload:** all side information needed for decoding is stored in the payload and counted in the compression ratio.
- **One setting:** the quantization bit depth.

## Measuring performance

- **Compression ratio (CR):** the size of the signal stored as 16-bit samples, divided by the size of the compressed payload, side information included.
- **Percent root-mean-square difference (PRD):** the root-mean-square reconstruction error as a percentage of the root-mean-square value of the original signal.

## Limitations

Because the waveform is preserved almost exactly, the compression ratio is much lower than that of lossy codecs. The codec is best suited to cases where fidelity matters more than size.

## Typical uses

Long-term ECG storage and transmission in wearable and remote-monitoring devices, where memory and bandwidth are limited but the waveform must stay essentially unchanged.

## Interface (preview)

```python
from near_lossless_codec import compress, decompress, compression_ratio, prd

payload = compress(x)                   # x: 1-D array, single-lead ECG
x_hat = decompress(payload)

print(compression_ratio(x, payload))    # 16-bit original size / payload size
print(prd(x, x_hat))                    # reconstruction error in percent
```

In this preview, these functions raise `NotImplementedError`. They show the interface that the released code will provide.

## Roadmap

- Release the implementation.
- Add usage examples and documentation.

## Reference

This codec implements the method of:

> S. Banerjee and G. K. Singh, "A new real-time lossless data compression algorithm for ECG and PPG signals," *Biomedical Signal Processing and Control*, vol. 79, p. 104127, 2023. doi: [10.1016/j.bspc.2022.104127](https://doi.org/10.1016/j.bspc.2022.104127)

If you use this method, please cite the original paper.

## Copyright

Copyright © 2026 Md Monzurul Islam Rabu. All rights reserved.

No permission is granted to use, copy, modify or redistribute the contents of this repository. An open-source license will be added when the code is released.
