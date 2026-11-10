---
name: qwen-colab-cli
description: Deploy a Qwen GGUF model to a Google Colab GPU runtime with the Colab CLI and llama.cpp, including health checks and an optional temporary tunnel.
---

# Qwen on Colab CLI

Use this skill when setting up or operating a Qwen `llama-server` deployment managed from Ubuntu WSL with Google's `colab` CLI.

1. Check `colab --help` and the installed CLI version before using flags. The CLI runs on Linux/macOS; use Ubuntu WSL from Windows. Authentication may require the user to complete a browser flow.
2. Create a named session and confirm its actual GPU before choosing a quantization or context size:

   ```bash
   colab new -s qwen-a100 --gpu A100
   colab status -s qwen-a100
   ```

   Check `nvidia-smi` in the runtime. Requested GPU type does not guarantee a specific VRAM size or successful allocation.
3. Download the chosen GGUF and, for multimodal models, its matching `mmproj` into `/content/models/`. Use `hf download` with explicit filenames; model repositories and filenames vary.
4. Use a Linux CUDA build of `llama-server` compatible with the runtime. Check `llama-server --version` and `--help`; set `LD_LIBRARY_PATH` if the release bundle requires separate CUDA libraries.
5. Start the server with the selected model, a context that fits the observed GPU, and `--n-gpu-layers`/`--fit` as supported by that build. Add `--mmproj` for vision and `--jinja` when the model needs its chat template or tool-call formatting. Keep logs under `/content` for diagnosis.
6. Keep API keys in environment variables; never commit or print real keys. Check `GET /health` and `/v1/models` locally before exposing the port. Describe API compatibility only for endpoints actually tested with this model and build.
7. If external access is requested, use a temporary tunnel such as Cloudflare Quick Tunnel, label its URL as temporary, and do not claim it survives a stopped runtime.
8. When finished, stop the session with `colab stop -s qwen-a100` and confirm it is gone with `colab sessions`.

For automated agent runs, prefer `colab exec -s <name> -f <script.py>` or piped non-interactive commands. Do not launch an interactive `colab console`/`repl` without a real terminal. If `llama-server.log` shows Qwen's `System message must be at the beginning` error, inspect the model chat template and patch only that observed case; a simple successful API request does not validate interactive Claude Code turns.

References: [Colab CLI](https://github.com/googlecolab/google-colab-cli), [llama.cpp server](https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md), [Hugging Face CLI](https://huggingface.co/docs/huggingface_hub/guides/cli).
