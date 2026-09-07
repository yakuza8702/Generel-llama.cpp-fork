# GenerelSchwerz llama.cpp fork — CUDA Docker builds

GitHub Actions workflow that builds Docker images of
[GenerelSchwerz/llama.cpp](https://github.com/GenerelSchwerz/llama.cpp) for
NVIDIA GPUs (RTX 3060 = CUDA arch `86`) and pushes them to GHCR.

Public repo → GitHub Actions standard runners are free (unlimited minutes).

## Build an image

1. Repo page → **Actions** → **Build GenerelSchwerz llama.cpp (CUDA docker)** → **Run workflow**
2. Pick:
   - **branch** (dropdown): `moe-cache`, `moe-cache-drafting`, `llama/main`, `llama/dev`, `beellama/main`
   - **cuda_arch**: leave `86` for RTX 3060 / 30-series (Ampere)
   - **extra_tag**: optional additional tag
3. Wait for the green run. The clone is `--depth 1` of the branch HEAD at run
   time, so every build grabs the **latest** commit — no cached sources.

## Pull

The first push creates the GHCR package as **private**. To pull without
login, make it public: your GitHub profile → **Packages** →
`generel-llama.cpp-fork` → Package settings → Change visibility → Public.

```bash
docker pull ghcr.io/yakuza8702/generel-llama.cpp-fork:moe-cache
```

Tags produced per run: `<branch-slug>` (e.g. `moe-cache`, `llama-main`),
`latest`, plus the optional extra tag.

## Run (12 GB VRAM, Qwen3.8-Flash-Next)

`moe-cache` branch flags come from the fork wiki's 12 GB VRAM setup:

```bash
docker run --gpus all --rm -it \
  -v /srv/dev-disk-by-uuid-b64105c0-07f7-4248-a6ee-186104f1aac0/ai-models:/models \
  ghcr.io/yakuza8702/generel-llama.cpp-fork:moe-cache \
  -m /models/UD-IQ4_XS/Qwen3.8-Flash-Next-UD-IQ4_XS-00001-of-00003.gguf \
  -ngl all -fit off -c 4096 -fa on -ctk q8_0 -ctv q8_0 -kvo \
  --load-mode none --moe-expert-cache-size 20 --cache-ram 0 \
  --host 0.0.0.0 --port 8080 --jinja
```

Start conservative (small `-c`, cache 20) and grow only if peak VRAM leaves
headroom. The wiki's measured 12 GB fits were on different models — always
watch `nvidia-smi` on first load.

## Files

- `.github/workflows/build-generel.yml` — workflow_dispatch build with branch dropdown
- `Dockerfile.generel` — CUDA 12.4 devel build → runtime image with `llama-server` etc.
