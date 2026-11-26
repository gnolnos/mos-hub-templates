# MOS Hub — gnolnos Templates

Docker templates for the **[MOS](https://forms-official.net) Hub** (My Own Service OS) for apps published by `gnolnos` that are not packaged upstream.

## Install

In MOS: **Settings → System Configuration → MOS Hub Settings** → add `https://github.com/gnolnos/mos-hub-templates` → **Refresh**. Then **Docker → MOS Hub** to deploy a template.

## Templates

| App | Port(s) | Image | Category | Notes |
|-----|---------|-------|----------|-------|
| [Gnolnos Portfolio](https://github.com/gnolnos/gnolnos-Portfolio) | 80 | `gnolnos/my-portfolio:latest` | Hosting | Static React portfolio (Nginx) |
| [Piper FastAPI Wrapper](https://github.com/gnolnos/piper-fastapi-wrapper) | 5002 | `gnolnos/piper-fastapi-wrapper:latest` | AI | REST TTS bridge → Piper Wyoming |
| [Wyoming Vietnamese ASR](https://github.com/gnolnos/Wyoming-Vietnamese-ASR) | 10301 / 8090 | `gnolnos/wyoming-vietnamese-asr:latest` | AI | Vietnamese STT; `MODE=wyoming` or `MODE=fastapi` |
| [WhisperX ASR Service](https://github.com/murtaza-nasir/whisperx-asr-service) | 9000 | `learnedmachine/whisperx-asr-service:latest` | AI | Word timestamps + diarization; needs `HF_TOKEN` |

### Compose stacks

| Stack | Service | Ports | Notes |
|-------|---------|-------|-------|
| [PolyVoice](compose/PolyVoice/README.md) | dubbing pipeline: WhisperX + WyViet-ASR + Piper + bridge + Ollama | 9000, 8090/10301, 5002, 10200, 11434 | Chain lồng tiếng đa ngôn ngữ; CPU, NVIDIA GPU, AMD/ROCm |

## Repository layout

```
docker/            # single-container templates (one JSON per app)
  GnolnosPortfolio.json
  PiperFastAPIWrapper.json
  WyomingVietnameseASR.json
  WhisperX.json
compose/           # multi-container stacks (one subfolder per app)
  PolyVoice/
    compose.yaml
    compose.gpu-nvidia.yaml
    compose.gpu-amd.yaml
    .env
    template.json
    README.md
images/            # PNG/JPG icons referenced by the JSON templates
  gnolnos-portfolio.png
  piper-tts-bridge.png
  whisperx.png
  wyoming-vietnamese-asr.png
  polyvoice.png
maintainer.json    # REQUIRED — the Hub ignores repos without this file
README.md
```

## Conventions

- Standard host path `/mnt/cache/appdata/<app>` for persistent data (portable across MOS installs).
- `maintainer.json` — `{"maintainer": "Phan Son Long", "donation": "https://github.com/sponsors/gnolnos"}`.
- Categories use exact MOS spellings (e.g. `AI`, `Hosting`).

## License

[MIT](LICENSE). Each template deploys the MIT-licensed project it references.
