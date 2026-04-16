---
description: "Encode a ProRes HDR master to Vimeo-optimized HEVC. Usage: /vimeo /path/to/source.mov"
---

Encode a ProRes 4444 XQ HDR master to HEVC Main 10 HLG for Vimeo upload — the highest quality delivery settings Vimeo supports.

The input file path is the argument to this command. If the user did not provide one, stop and ask for it.

## Pre-flight

1. **Verify the source exists:**
   ```bash
   test -f "$INPUT" || { echo "ABORT: file not found"; exit 1; }
   ```

2. **Probe the source and display a summary:**
   ```bash
   ffprobe -hide_banner -i "$INPUT" 2>&1
   ```
   Confirm the source is ProRes (ap4h/ap4x), HDR (bt2020/arib-std-b67), and has video. Print a one-line summary: resolution, fps, duration, codec, color info, audio format. If it's not ProRes HDR, warn the user but don't abort — they may have a valid reason.

3. **Check available disk space** on the output volume. Warn if less than 10 GB free.

## Encode

Output filename: same directory and base name as input, with ` - Vimeo.mp4` suffix.

```bash
ffmpeg -i "INPUT" \
  -map 0:v:0 -map 0:a:0 \
  -c:v libx265 \
  -profile:v main10 \
  -pix_fmt yuv420p10le \
  -preset slow \
  -crf 14 \
  -color_primaries bt2020 \
  -color_trc arib-std-b67 \
  -colorspace bt2020nc \
  -color_range tv \
  -x265-params "colorprim=bt2020:transfer=arib-std-b67:colormatrix=bt2020nc:range=limited:chromaloc=2:atc-sei=18" \
  -tag:v hvc1 \
  -c:a aac -b:a 320k -ar 48000 \
  -movflags +faststart \
  "OUTPUT - Vimeo.mp4"
```

Run the encode via Bash. This will take a while — use a generous timeout (10 minutes) or `run_in_background: true` for long files. Monitor progress and report when complete.

### Settings rationale
- **HEVC Main 10** — Vimeo's recommended HDR codec; hardware-decoded on Apple devices
- **CRF 14 / slow** — highest practical quality for a platform that re-encodes. Lower CRF gives diminishing returns and wastes upload bandwidth
- **HLG signaled twice** — `-color_*` flags write the MP4 `colr` atom; the matching `colorprim/transfer/colormatrix` inside `-x265-params` writes the HEVC VUI inside the bitstream. Vimeo's transcoder reads the bitstream first; if VUI is missing it tone-maps to SDR (this was the cause of past "Vimeo dimmed my HDR" uploads)
- **`atc-sei=18`** — Alternative Transfer Characteristics SEI (value 18 = HLG/arib-std-b67). Belt-and-braces hint to downstream pipelines that the content is HLG even if they only parse the matrix as bt2020nc
- **`-color_range tv` + `range=limited`** — explicit limited range on both container and bitstream; ambiguity here also triggers Vimeo SDR fallback
- **hvc1 tag** — required for Apple HDR playback pipeline
- **AAC 320k / 48kHz** — Vimeo's recommended audio spec
- **faststart** — moov atom at front for streaming/progressive upload
- **yuv420p10le** — strips alpha from ProRes 4444 XQ source; 10-bit 4:2:0 is Vimeo's HDR maximum
- **`-map 0:v:0 -map 0:a:0`** — explicitly map only video+audio; skips QuickTime timecode/data tracks that ffmpeg can't mux into MP4

## Post-encode

1. **Report results:**
   ```bash
   ffprobe -hide_banner -i "OUTPUT" 2>&1
   du -h "OUTPUT"
   ```
   Print: output file size, average video bitrate, duration, and confirm HDR metadata is present (bt2020/arib-std-b67 in the output probe).

2. **Sanity check:** If output is larger than the input, something went wrong — warn the user.

3. **Print upload reminder:** "Ready for Vimeo upload. HDR will be preserved — verify in Vimeo's player after processing."
