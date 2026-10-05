# Home AI Lab

A practical project for learning Linux, Git, local AI, and HTTP APIs. The first application goal is a knowledge assistant that answers questions from my own notes and identifies its sources. A later goal is a simpler interface to my smart-home devices.

**Current milestone:** local model inference and an HTTP generation request verified. Updated October 5, 2026 (UTC).

## Progress

| Milestone | Status |
| --- | --- |
| Ubuntu under WSL2 and NVIDIA GPU visibility | Verified |
| GitHub repository and SSH authentication | Verified; API evidence pushed in commit `94fda9d` |
| Ollama installation and model download | Verified |
| Local text generation with full GPU placement | Verified September 30, 2026 |
| HTTP generation and saved JSON evidence | Verified October 5, 2026 UTC |
| Knowledge assistant using personal documents | Planned |
| Smart-home integration | Future goal |
| Docker and Kubernetes | Later learning stages |

## Environment

| Component | Recorded configuration |
| --- | --- |
| Laptop | ASUS ROG Zephyrus M15 / GU502LU |
| CPU | Intel Core i7-10750H |
| RAM | 40 GB |
| GPU | NVIDIA GeForce GTX 1660 Ti, 6 GB VRAM |
| Host | Windows 11 Pro for Workstations |
| Linux | Ubuntu 24.04 LTS under WSL2 |
| Ollama | 0.35.0, verified September 30, 2026 |
| Model | `llama3.2:3b`, ID `a80c4f17acd5` |
| Model registry metadata | 3.21B parameters, `Q4_K_M` quantization, approximately 2.0 GB download |

Ollama runs inside Ubuntu. The CLI and HTTP client use the local model server; the tested endpoint is `http://localhost:11434/api/generate`. Generated responses are saved in the repository's `evidence` directory.

## Setup and usage

The recorded setup assumes WSL2, Ubuntu, and working NVIDIA GPU access are already available. Enter each command separately. Installation commands below are for a new setup; an existing working installation can proceed directly to the tests. The installer retrieves the current release, which may differ from the version recorded above.

In **Windows PowerShell**, enter Ubuntu:

```powershell
wsl -d Ubuntu-24.04
```

In **Ubuntu**, open the existing project:

```bash
cd ~/home-ai-lab
```

For a new installation, install the extraction dependency and run the official Ollama installer:

```bash
sudo apt update
sudo apt install -y curl zstd
curl -fsSL https://ollama.com/install.sh | sh
```

Verify GPU visibility, the executable, and the model list:

```bash
nvidia-smi
ollama --version
ollama ls
```

### CLI inference test

In **Ubuntu**:

```bash
ollama run llama3.2:3b
```

At the **Ollama chat prompt**, enter:

```text
Explain DNS to a new help desk technician in three sentences.
```

After the response, enter `/bye`. Back in **Ubuntu**, check placement while the model is still loaded:

```bash
ollama ps
```

The September 30 test returned a DNS explanation and reported:

| Field | Observed value |
| --- | --- |
| Loaded model | `llama3.2:3b` |
| Loaded size | 2.6 GB |
| Processor placement | `100% GPU` |
| Context | 4096 tokens |

`100% GPU` means the model was loaded entirely into GPU memory. This field reports model placement, rather than instantaneous GPU utilization. Download size and loaded size describe different states of the model.

### HTTP inference test

In **Ubuntu**, from the project directory:

```bash
mkdir -p evidence
curl -fsS http://localhost:11434/api/generate -d '{"model":"llama3.2:3b","prompt":"Explain DHCP in two sentences.","stream":false}' -o evidence/api-test.json
python3 -m json.tool evidence/api-test.json
```

Setting `stream` to `false` returns a single JSON response. The completed test generated a two-sentence DHCP explanation and returned `done: true` with `done_reason: "stop"`. The saved response parsed successfully with Python's JSON tool.

Evidence: [saved API response](evidence/api-test.json), pushed to GitHub in commit `94fda9d` on October 4, 2026 (America/New_York).

| Measurement | Observed value |
| --- | --- |
| Response timestamp | `2026-10-05T03:00:04.643269225Z` |
| Total server duration | 5.1096 seconds |
| Model load duration | 4.0567 seconds |
| Prompt tokens | 32 |
| Prompt evaluation duration | 0.2051 seconds |
| Generated tokens | 72 |
| Token generation duration | 0.8449 seconds |
| Derived generation rate | Approximately 85.2 tokens/second |

Ollama reports durations in nanoseconds. The generation rate is `72 / (844912000 / 1000000000)`. It covers token generation only; the total duration also includes loading and other processing. These results describe one short request. The GPU-placement observation above comes from the earlier CLI test.

## Troubleshooting recorded

| Issue | Resolution |
| --- | --- |
| Installer stopped because `zstd` was missing | Installed `zstd` with Ubuntu's package manager, reran the installer, and verified Ollama. |
| Terminal paste introduced extra characters | Entered commands separately; corrected malformed commands such as `ollama list~`. |
| Unclear which shell was active | Used the prompt to distinguish PowerShell, Ubuntu, and the Ollama chat. |
| API evidence file initially absent | Created `evidence`, executed the HTTP request, and checked the saved JSON. |

## Next milestones

1. Publish this README in GitHub; the saved API response is already pushed.
2. Build a small knowledge assistant using a defined set of notes, with source references and a clear response when the notes do not contain an answer.
3. Evaluate it with questions whose answers are present and absent in those notes.
4. Inventory smart-home devices and assess compatible integrations.
5. Add Docker or Kubernetes when there is a working application that benefits from them.

The current implementation provides local model inference through Ollama's existing CLI and API. Document retrieval and smart-home control remain future work.

## References

- [Project repository](https://github.com/amiralmo/home-ai-lab)
- [Ollama Linux installation](https://docs.ollama.com/linux)
- [Llama 3.2 3B model](https://ollama.com/library/llama3.2:3b)
- [Ollama GPU-placement explanation](https://docs.ollama.com/faq)
- [Ollama generation API](https://docs.ollama.com/api/generate)
- [Microsoft WSL commands](https://learn.microsoft.com/en-us/windows/wsl/basic-commands)
