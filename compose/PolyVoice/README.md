# PolyVoice — chain lồng tiếng đa ngôn ngữ (dubbing pipeline)

Stack compose cho **MOS** (My Own Service OS) chạy chain xử lý giọng nói đầy đủ, triển khai được trên bất kỳ máy MOS trắng nào (CPU, NVIDIA GPU, AMD GPU).

## Các service

| Service | Image | Port | Vai trò |
|---------|-------|------|---------|
| `whisperx` | learnedmachine/whisperx-asr-service | 9000 | ASR đa ngôn ngữ + word timestamps + diarization (OpenAI-compatible /v1/audio/transcriptions + /docs) |
| `wyviet-asr` | gnolnos/wyoming-vietnamese-asr | 8090 (REST) / 10301 (Wyoming) | ASR tiếng Việt chuyên dụng, model RNNT Zipformer, auto-download từ HuggingFace |
| `piper` | rhasspy/wyoming-piper | 10200 | Piper Wyoming TCP server (hàng chục giọng, 44+ ngôn ngữ) |
| `piper-bridge` | gnolnos/piper-fastapi-wrapper | 5002 | REST TTS bridge đơn giản: GET /health, GET /voices, POST /tts → audio/wav |
| `ollama` | ollama/ollama | 11434 | LLM cục bộ: dịch thoại, viết kịch bản, xử lý văn bản trước khi TTS |

## Flow (dubbing pipeline)

```
Audio (phim/khách hàng)
   └─▶ WhisperX  → lời thoại + timeline (phụ đề)
   └─▶ WyViet    → phụ đề tiếng Việt (nếu nguồn Việt/modern)
        │
        ▼
 Ollama (dịch lời thoại → ngôn ngữ đích, giữ giọng điệu)
        │
        ▼
 piper-bridge → piper → TTS audio (giọng Việt/đa ngôn ngữ)
```

Ứng dụng thực tế: dubbing anime/kids-animation, lồng tiếng phim, đọc tin hằng ngày, podcast tự động.

## Triển khai

Copy cả 4 file vào thư mục `/boot/config/system/docker/compose/polyvoice/` rồi deploy bằng MOS Hub, hoặc trực tiếp:

```bash
docker compose up -d
```

Mặc định chạy **CPU** — port/path đều qua biến trong `.env` (`APP_DATA=/mnt/cache/appdata/polyvoice`, PUID/PGID mặc định MOS).

### GPU — NVIDIA
Ollama `:latest` tự dò NVIDIA. Để WhisperX dùng GPU:

```bash
docker compose -f compose.yaml -f compose.gpu-nvidia.yaml up -d
```

### GPU — AMD / ROCm
Ollama chuyển sang image `rocm` + gắn `/dev/kfd` + `/dev/dri`:

```bash
docker compose -f compose.yaml -f compose.gpu-amd.yaml up -d
```

> Lưu ý: WhisperX chưa hỗ trợ AMD ROCm chính thức — trên máy AMD giữ `whisperx` ở `DEVICE=cpu`. LLM (Ollama) tận dụng được GPU AMD.

## Vo ngữ (voice)

- Danh sách giọng Piper: `./piper --list-voices` hoặc qua bridge: `GET http://<host>:5002/voices`.
- Mặc định `vi_VN-vais1000-medium`. Đổi trong `.env` (`PIPER_VOICE`, `PIPER_LANGS`).
- Giọng được tải về `${APP_DATA}/piper/data` (tiết kiệm khi đổi máy / offline).

## API nhanh

```bash
# TTS (streaming wav)
curl -X POST http://<host>:5002/tts -H "Content-Type: application/json" \
  -d '{"text":"Xin chào, tôi là Jarvis.","voice":"vi_VN-vais1000-medium"}' --output out.wav

# Transcription tiếng Việt
curl -X POST http://<host>:8090/transcribe -F "file=@speech.wav"

# Transcription đa ngôn ngữ (WhisperX, OpenAI-compatible)
curl -X POST http://<host>:9000/v1/audio/transcriptions -F "file=@speech.wav"
```

## License
MIT — © 2026 Phan Son Long (gnolnos). Các image phụ thuộc (WhisperX, Ollama, Piper) giữ license riêng của chúng.
