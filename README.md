# Hugging Face Docker Workspace

This repository provides a Docker Compose environment for running Hugging Face
Transformers with PyTorch GPU support.

## Requirements

- Docker
- Docker Compose
- NVIDIA GPU driver
- NVIDIA Container Toolkit
- A host dataset directory at `/mnt/datasets` if you want to use the mounted
  datasets path

## Services

### `huggingface`

The Compose file starts one long-running container:

- Image: `huggingface/transformers-pytorch-gpu:latest`
- Container name: `huggingface`
- Working directory: `/workspace`
- GPU access: `gpus: all`
- Shared memory: `128gb`
- IPC mode: `host`
- Command: `sleep infinity`

The container is intended to stay running so you can enter it interactively and
run notebooks, scripts, training jobs, or inference commands.

## Directory Layout

```text
.
├── docker-compose.yaml
├── README.md
└── workspace/              # Mounted to /workspace inside the container
```

The `workspace` directory is mounted into the container at `/workspace`. Put your
training scripts, notebooks, model code, and experiment files there.

## Cache and Data Mounts

The environment uses these Hugging Face cache paths:

```text
HF_HOME=/workspace/.cache/huggingface
TRANSFORMERS_CACHE=/workspace/.cache/huggingface/hub
HF_DATASETS_CACHE=/workspace/.cache/huggingface/datasets
```

Compose also creates a named Docker volume:

```text
hf-hub-cache
```

This keeps downloaded models and datasets available across container restarts.

The host path `/mnt/datasets` is mounted into the container as:

```text
/datasets
```

Create `/mnt/datasets` on the host or edit `docker-compose.yaml` if your datasets
are stored somewhere else.

## Hugging Face Token

The Compose file reads `HF_TOKEN` from your shell environment:

```bash
export HF_TOKEN=your_huggingface_token
```

This is optional, but required for private models, gated models, or authenticated
Hub access.

## Usage

Start the container:

```bash
docker compose up -d
```

Open an interactive shell:

```bash
docker exec -it huggingface bash
```

Run a quick GPU check inside the container:

```bash
python -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0))"
```

Stop the container:

```bash
docker compose down
```

Stop the container and remove the named cache volume:

```bash
docker compose down -v
```

## Notes

- The image tag uses `latest`, so builds may change over time. Pin a specific
  image tag in `docker-compose.yaml` if you need reproducible environments.
- `shm_size: "128gb"` is useful for large dataloaders, but it should not exceed
  what your host can reasonably provide.
- `ipc: host` is commonly used for PyTorch multiprocessing workloads, but it
  reduces container isolation.
