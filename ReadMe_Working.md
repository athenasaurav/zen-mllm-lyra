# Lyra (Zen) - Working Installation & Launch Guide

This guide documents the **actual working steps** to set up and run the Lyra (Zen) multi-modal model, as tested on a multi-GPU server. It is based on real shell history and is intended to help you reproduce a successful environment and launch.

---

## 1. Clone the Repository
```bash
git clone https://github.com/athenasaurav/zen-mllm-lyra.git
cd zen-mllm-lyra/
```

## 2. Set Up Conda Environment
```bash
conda create -n lyra python=3.10 -y
conda activate lyra
```

## 3. Install Python Dependencies
Install core and required packages:
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

**Note:**
- `flash-attn` must be installed separately due to special build flags:
    ```bash
    pip install flash-attn==2.6.3 --no-build-isolation --no-cache-dir
    ```

## 4. Install Git LFS (for model weights)
```bash
sudo apt-get install -y git-lfs && git lfs install
```

## 5. Download Model Weights and Resources
Create necessary directories:
```bash
mkdir -p model_zoo/vision
mkdir -p model_zoo/audio/vocoder
mkdir -p model_zoo/LLM
```

### Download Vision Encoder
```bash
cd model_zoo/vision
git clone https://huggingface.co/zszhong/Lyra_Qwen2VL_2B_ViT Qwen2VL_2B_ViT
cd ../..
```

### Download Audio Encoder (Whisper)
```bash
cd model_zoo/audio/
git clone https://huggingface.co/openai/whisper-large-v3-turbo
cd ../..
```

### Download Vocoder
```bash
cd model_zoo/audio/vocoder/
wget https://dl.fbaipublicfiles.com/fairseq/speech_to_speech/vocoder/code_hifigan/mhubert_vp_en_es_fr_it3_400k_layer11_km1000_lj/g_00500000
wget https://dl.fbaipublicfiles.com/fairseq/speech_to_speech/vocoder/code_hifigan/mhubert_vp_en_es_fr_it3_400k_layer11_km1000_lj/config.json
cd ../../../..
```

### Download LLM
```bash
cd model_zoo/LLM
git clone https://huggingface.co/zszhong/Lyra_Qwen2VL_2B_LLM Qwen2VL_2B_LLM
cd ../..
```

### Download Model Checkpoint
```bash
mkdir -p work_dirs/Lyra
cd work_dirs/Lyra
git clone https://huggingface.co/zszhong/Lyra_Mini_3B
cd ../../
```

---

## 6. Upgrade/Install Additional Packages
Some additional steps may be required for compatibility:
```bash
pip install pip==24.0
pip install fairseq==0.12.2
pip install --upgrade gradio
```

---

## 7. Launch the System

### Use Two `tmux` Sessions (Recommended)

#### **Session 1: Controller**
```bash
tmux new -s controller
conda activate lyra
python -m lyra.serve.controller --host 0.0.0.0 --port 10000
```

#### **Session 2: Model Worker**
```bash
tmux new -s model
conda activate lyra
CUDA_VISIBLE_DEVICES='0,1' python -m lyra.serve.model_worker \
    --host 0.0.0.0 \
    --controller http://localhost:10000 \
    --port 40000 \
    --worker http://localhost:40000 \
    --model-path work_dirs/Lyra/Lyra_Mini_3B \
    --model-lora-path work_dirs/Lyra/Lyra_Mini_3B/speech_lora
```
- **Note:** Adjust `CUDA_VISIBLE_DEVICES` as needed for your GPU setup.

#### **Session 3: Gradio Web Server**
You can run this in a new terminal or tmux session:
```bash
tmux new -s gradio
conda activate lyra
python -m lyra.serve.gradio_web_server \
    --controller http://localhost:10000 \
    --model-list-mode reload \
    --port 443
```
- The web UI will be available at `https://<your-server>:443` (or `http://` if not using SSL). also you can use Share URL from gradio.

---

## 8. Access the Web UI
Open your browser and go to the server's address and port you specified (default: 443) or use Share URL.

---

## 9. Notes & Troubleshooting
- **Multi-GPU:** The model worker is launched with two GPUs (`CUDA_VISIBLE_DEVICES='0,1'`). Adjust as needed.
- **Model Paths:** Ensure all model and lora paths are correct and downloaded.
- **SSL:** If you want to use SSL, provide `--ssl-certfile` and `--ssl-keyfile` to the web server command.
- **Dependencies:** If you encounter missing packages, install them as needed using `pip`.
- **Gradio Version:** If you have UI issues, try upgrading Gradio as shown above.

---

## 10. Example Directory Structure
```
zen-mllm-lyra/
├── model_zoo/
│   ├── vision/
│   │   └── Qwen2VL_2B_ViT/
│   ├── audio/
│   │   ├── whisper-large-v3-turbo/
│   │   └── vocoder/
│   │       ├── g_00500000
│   │       └── config.json
│   └── LLM/
│       └── Qwen2VL_2B_LLM/
├── work_dirs/
│   └── Lyra/
│       └── Lyra_Mini_3B/
├── lyra/
├── scripts/
├── ...
```

---

## 11. References
- This guide is based on actual working commands and may differ from the original README for clarity and reproducibility.
- For more details, see the original [Lyra README](README.md) or [project page](https://lyra-omni.github.io/). 