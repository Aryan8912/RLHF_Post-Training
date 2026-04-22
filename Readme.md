# RLHF Post Training with GRPO
# RLHF Post-Training Pipeline — Complete Colab Notebooks

A complete, runnable implementation of the RLHF alignment pipeline using GPT-2 (117M params) and HuggingFace libraries. Every notebook runs on a **free Colab T4 GPU** in under 10 minutes.

---

## Notebooks

| # | File | Stage | Key concepts |
|---|------|-------|-------------|
| 01 | `01_SFT_LoRA.ipynb` | Supervised Fine-Tuning | LoRA, cross-entropy loss, instruction format |
| 02 | `02_RewardModel.ipynb` | Reward Model Training | Bradley-Terry loss, scalar reward head, preference data |
| 03 | `03_PPO.ipynb` | PPO RL Fine-Tuning | PPOTrainer, KL penalty, policy vs reference model |
| 04 | `04_GRPO_DPO.ipynb` | GRPO + DPO alternatives | Group relative advantages, direct preference optimization |
| 05 | `05_Eval.ipynb` | Evaluation & Alignment | MT-Bench scoring, win rates, sycophancy probes |

---

## Run order

```
01_SFT_LoRA → 02_RewardModel → 03_PPO → 04_GRPO_DPO → 05_Eval
```

Each notebook saves checkpoints that the next one can load. For the full pipeline, run them in order.

---

## Requirements

All dependencies installed inside each notebook:
```
transformers  datasets  peft  trl  accelerate  bitsandbytes
```

**Runtime**: GPU (T4) recommended. All notebooks work on free Colab.

---

## Pipeline architecture

```
Pretrained          SFT                 Reward Model         PPO / GRPO / DPO     Eval
base model    →   fine-tune    →     train on pairwise  →   RL fine-tune      →  benchmark
(GPT-2)          (LoRA + CE)         preferences (BT)       (KL-constrained)     + iterate
```

### Key design choices

**SFT (01)**
- LoRA rank=8, alpha=32, targeting `c_attn` + `c_proj`
- Only ~0.3% of parameters trained
- 3 epochs, cosine LR, effective batch 8

**Reward Model (02)**  
- GPT-2 backbone + linear scalar head
- Bradley-Terry loss: `-log σ(r_chosen - r_rejected)`
- Eval metric: accuracy (% pairs where chosen > rejected)

**PPO (03)**
- `AutoModelForCausalLMWithValueHead` from TRL
- KL target = 6.0 with adaptive β coefficient
- Clip range ε = 0.2

**GRPO (04)**
- G=4 samples per prompt, group-relative advantage normalisation
- No separate value network
- Same KL penalty structure as PPO

**DPO (04)**
- `DPOTrainer` from TRL with β=0.1
- No reward model, no RL loop
- Directly optimises preference pairs

---

## Connecting to your own models

Replace `MODEL_NAME = "gpt2"` with any HuggingFace model that fits your GPU:

| Model | Params | VRAM needed |
|-------|--------|-------------|
| `gpt2` | 117M | ~2 GB |
| `gpt2-medium` | 345M | ~4 GB |
| `gpt2-large` | 774M | ~8 GB |
| `distilgpt2` | 82M | ~1.5 GB |
| `facebook/opt-125m` | 125M | ~2 GB |

For 7B+ models: use QLoRA (4-bit) via `bitsandbytes` — replace `load_in_4bit=True` in the model loading step.
