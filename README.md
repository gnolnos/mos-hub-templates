# mos-hub-templates

Container templates for MOS Hub and Unraid, following the [ich777/mos-templates](https://github.com/ich777/mos-templates) convention.

## Structure

```
docker/   → MOS Hub JSON templates
unraid/   → Unraid XML templates (dockerMan templates-user)
```

## Templates

| Template | Host | Notes |
|----------|------|-------|
| `docker/WhisperX.json` | HuyenVu (MOS / BlackTortoise) | CPU mode (DEVICE=cpu, COMPUTE_TYPE=int8, model=small). This box has only Intel HD Graphics 500 — no NVIDIA. |
| `unraid/whisperx-asr.xml` | ThanhLong (Unraid / AzureDragon) | GPU mode (DEVICE=cuda, COMPUTE_TYPE=float16, model=large-v3, `--gpus all`), RTX 3060. |

## WhisperX ASR Service

- Image: `learnedmachine/whisperx-asr-service:latest`
- Source: https://github.com/murtaza-nasir/whisperx-asr-service
- API: port `9000`, OpenAI-compatible `/v1/audio/transcriptions`, diarization, word timestamps.
- **HF_TOKEN is REQUIRED for speaker diarization** — generate a read token and accept the three pyannote model agreements:
  - https://huggingface.co/pyannote/speaker-diarization-community-1
  - https://huggingface.co/pyannote/segmentation-3.0
  - https://huggingface.co/pyannote/speaker-diarization-3.1
- GPU tag `:latest` = cu126 (any NVIDIA 10xx–40xx, A/H series). RTX 50xx → use `:blackwell`. CPU-only → works with `DEVICE=cpu`, slower.

## Capability notes

- **ThanhLong** (RTX 3060 12GB): run `large-v3` + diarization. Ideal for full-pipeline anime dubbing (multilingual ASR: EN/JA/ZH → Vietnamese TTS).
- **HuyenVu** (J3455, Intel iGPU only): no CUDA. Run `small` model, `DEVICE=cpu`, `int8` — usable for short clips / low-throughput ASR, not production diarization.

## Usage (Unraid)

Drop `unraid/whisperx-asr.xml` into `/boot/config/plugins/dockerMan/templates-user/`, then it appears in the Unraid Dockers page under "Add Container" (template = whisperx-asr).

## Usage (MOS)

Import `docker/WhisperX.json` via the MOS Hub UI.
