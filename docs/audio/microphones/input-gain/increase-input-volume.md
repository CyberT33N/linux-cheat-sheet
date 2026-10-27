# Increase Microphone Input Volume On Linux (PipeWire) — Canonical Blueprint

This document is the canonical, self-contained blueprint for diagnosing and fixing a too-quiet microphone input on a Linux system running PipeWire/WirePlumber, **without destroying audio quality**. It is written device-agnostically: every step is parameterized (`<card>`, `<node-id>`, `<factor>`) and can be applied to any USB or built-in microphone. The concrete LCS Audio / Medeli case at the end serves as the worked example.

## Core Rule: Measure First, Then Gain

A blind boost of the input slider beyond 100% (for example 130%) is **not** an acceptable solution: it destroys signal quality. Every gain increase must be derived from a measured level and a proven clipping headroom budget.

The three governing constraints:

1. Check and exhaust the **hardware (analog) gain** before applying any digital gain.
2. Apply digital gain **only within the measured headroom** (target: resulting peak ≤ -6 dBFS).
3. Always **verify by readback** and always **document the undo path**.

## Gain-Stage Architecture

```text
microphone capsule → analog preamp (ALSA hardware control) → ADC → digital domain (PipeWire float)
```

- The analog preamp is the only stage that improves the signal-to-noise ratio (SNR); it lives in the ALSA mixer of the capture card, typically in dB.
- Digital gain in PipeWire scales **signal and noise equally**, so the SNR does not change — but it is lossless up to the clipping boundary because PipeWire processes internally in 24/32-bit float.
- Therefore: push the analog stage to its usable maximum first; use digital gain only for the remaining distance, bounded by measured headroom.

## Prerequisites

- PipeWire + WirePlumber (commands `wpctl`, `pactl` available)
- ALSA utilities (`amixer`)
- `pw-record` (PipeWire tools) for capture
- Python 3 with the standard-library modules `wave` and `audioop` for level analysis (see the compatibility note in step 3)

## Step 1 — Locate The Capture Card

```shell
cat /proc/asound/cards
cat /proc/asound/pcm
```

Identify the card that owns the microphone. A capture-only PCM device (no playback direction) is the typical signature of a USB microphone.

```text
01-00: USB Audio : USB Audio : capture 1
```

## Step 2 — Read Hardware Limits And Current State

Inspect the device identity and the mixer capabilities:

```shell
cat /proc/asound/<card>/usbid
cat /proc/asound/<card>/stream0
cat /proc/asound/<card>/usbmixer
amixer -c <card> controls
```

Find the capture-gain control (usually `Mic Capture Volume`) and read its **numid**, then read its exact current value:

```shell
amixer -c <card> cget numid=<gain-numid>
```

Interpretation:

- The control header shows the raw range and the dB range (for example `min=0,max=32768` spanning `-96.00 dB … +32.00 dB`).
- `values=` at the raw maximum means the analog preamp is already fully exhausted — **the GUI showing 100% is truthful**, and no hidden reserve exists in hardware.
- A common wrong hypothesis is that the GUI maps 100% to the middle of the hardware range. **Disprove it by measurement** before acting on it; this prevents the classic mis-fix "turn the slider up further although it is already at the limit".

## Step 3 — Measure The Actual Input Level

Record silence (baseline) and speech, then compute peak and RMS in dBFS:

```shell
timeout 3 pw-record --rate 48000 --channels 1 --format s16 "${TMPDIR:-/tmp}/mic_before.wav"
# speak a normal sample sentence when prompted
timeout 6 pw-record --rate 48000 --channels 1 --format s16 "${TMPDIR:-/tmp}/mic_speech.wav"
```

Level analysis script (peak/RMS in dBFS):

```python
import audioop
import math
import sys
import wave


def analyze(path: str) -> None:
    with wave.open(path, "rb") as wav:
        sample_width = wav.getsampwidth()
        frames = wav.readframes(wav.getnframes())

    full_scale = 2 ** (8 * sample_width - 1) - 1
    peak = audioop.max(frames, sample_width)
    rms = audioop.rms(frames, sample_width)

    def to_dbfs(value: int) -> str:
        if value <= 0:
            return "-inf"
        return f"{20 * math.log10(value / full_scale):.1f} dBFS"

    print(f"{path}: peak {peak}/{full_scale} = {to_dbfs(peak)}, rms {rms}/{full_scale} = {to_dbfs(rms)}")


if __name__ == "__main__":
    analyze(sys.argv[1])
```

```shell
python3 analyze_level.py "${TMPDIR:-/tmp}/mic_speech.wav"
```

> Compatibility note: `audioop` was removed in Python 3.13. On such runtimes compute peak and RMS from the decoded samples (for example via the `array` module) with the same dBFS formula `20 · log10(value / full_scale)`.

Reference bands for speech:

| Metric | Healthy target | Typical too-quiet mic |
|--------|----------------|-----------------------|
| Speech peak | ≈ -6 … -3 dBFS | ≈ -24 dBFS or below |
| Speech RMS | ≈ -20 … -25 dBFS | ≈ -47 dBFS |

If the hardware gain is already at maximum (step 2) and the measured level is far below the target band, the cause is the weak analog path of the device itself — the remaining reserve exists only in the digital domain.

## Step 4 — Find The PipeWire Gain Node

```shell
wpctl status
```

Note the source node id (`<node-id>`) of the microphone. Confirm the node supports decibel volume (`DECIBEL_VOLUME` flag in `pactl list sources`). `wpctl` works with **linear factors**: `1.00` = 0 dB.

## Step 5 — Derive The Boost From The Headroom Budget

Conversion between factor and dB:

```text
dB = 20 · log10(factor)        factor = 10^(dB / 20)
```

Budget rule: `measured_peak_dBFS + planned_boost_dB ≤ -6 dBFS`.

Example derivation: measured speech peak -24.3 dBFS → a +12 dB boost lands the peaks at -12.3 dBFS, leaving 12 dB of clipping headroom. +12 dB corresponds to the linear factor `10^(12/20) ≈ 4.0`.

Why this specific digital boost is quality-preserving:

1. **SNR unchanged** — signal and noise scale by the same factor.
2. **No resolution loss** — the multiplication happens inside PipeWire's float domain with sufficient bit depth.
3. **No clipping** — proven by the pre-measurement, not asserted.

## Step 6 — Apply And Verify

```shell
wpctl set-volume <node-id> <factor>
wpctl get-volume <node-id>
```

Setting without the `get-volume` readback is incomplete: only the readback proves the target state on the exact node. Re-measure with a fresh recording if you want end-to-end confirmation; expected speech peak = previous peak + boost.

## Undo

```shell
wpctl set-volume <node-id> 1.0
```

## Persistence

WirePlumber stores per-node volumes; the setting survives reboots without touching any `~/.config` file.

## If Noise Becomes Noticeable After The Boost (Optional, Root)

Digital gain raises the noise floor by the same amount as the signal. If that is unacceptable, the governed path is a filter chain (for example RNNoise gate + limiter) through PipeWire's `filter-chain` module. This requires LADSPA plugin packages (for example `lsp-plugins-ladspa` or `calf-plugins`) installed via a root-privileged, interactive-sudo package installation — a deliberate, user-approved step, not a silent side effect.

## Longer-Term Alternative

A separate USB audio interface with real analog inputs typically provides more clean analog gain than an entry-level microphone's built-in chip. If the microphone itself has a line/mic combo input on such an interface, moving it there is the quality-optimal fix.

## Worked Example — LCS Audio / Medeli USB Microphone

Environment: Ubuntu Linux, kernel 6.17, zsh, PipeWire 1.4.7 + WirePlumber 1.4.7 (KDE/Plasma).

| Property | Value |
|----------|-------|
| Card name | "USB LCS AUDIO" (card 1, full-speed USB) |
| USB vendor:product | `0a67:ffff` (Medeli Electronics Co., Ltd) |
| Capture format | S24_3LE, 1 channel (mono), 192000 Hz |
| Hardware gain control | `Mic Capture Volume` (numid 3), range -96.00 dB … +32.00 dB |
| Measured hardware state | `values=32768,32768` = +32 dB = **maximum**; capture switch on |
| PipeWire node | source "LCS Audio Pro", node id 81, `DECIBEL_VOLUME`, volume 1.00 / 0.00 dB |
| Measured levels | silence peak ≈ -34.5 dBFS; speech peak **-24.3 dBFS**; speech RMS -47.4 dBFS |
| Diagnosis | analog path exhausted; weakness is intrinsic to the chip's analog stage |
| Applied fix | `wpctl set-volume 81 4.0` (+12 dB), readback `Volume: 4.00` |
| Result | expected speech peaks ≈ -12.3 dBFS, 12 dB clipping headroom, no quality loss; user confirmed the outcome |

Chronological command sequence of the executed case (card 1, node 81):

```shell
cat /proc/asound/cards
cat /proc/asound/pcm
cat /proc/asound/card1/stream0
cat /proc/asound/card1/id
cat /proc/asound/card1/usbid
cat /proc/asound/card1/usbmixer
amixer -c 1 controls
amixer -c 1 cget numid=3
amixer -c 1 cget numid=2
pactl list sources
wpctl status
timeout 3 pw-record --rate 48000 --channels 1 --format s16 "${TMPDIR:-/tmp}/mic_before.wav"
timeout 6 pw-record --rate 48000 --channels 1 --format s16 "${TMPDIR:-/tmp}/mic_speech.wav"
python3 analyze_level.py "${TMPDIR:-/tmp}/mic_speech.wav"
wpctl get-volume 81
wpctl set-volume 81 4.0
wpctl get-volume 81
```

## Decision Rules (Summary)

| Rule | Scope |
|------|-------|
| No blind software boost beyond 100% without level measurement and headroom proof | every gain change |
| Measure (mixer read + peak/RMS), then set, then verify by readback | gain staging |
| Check and exhaust hardware gain before any digital boost | diagnosis order |
| Always document the undo path | reversibility |
| Recordings capture audio — treat them as side effects that need explicit consent | terminal dispatches |