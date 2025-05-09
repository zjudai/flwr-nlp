# FlowerTune LLM on General NLP/Medical/Finance

This directory conducts federated instruction tuning with pretrained language models on the Flowertune-llm benchmark, including a general NLP dataset [vicgalle/alpaca-gpt4](https://huggingface.co/datasets/vicgalle/alpaca-gpt4), a Medical dataset, and a Finance dataset.
We use [Flower Datasets](https://flower.dev/docs/datasets/) to download, partition, and preprocess the dataset.
Flower's Simulation Engine is used to simulate the LLM fine-tuning process in a federated way,
which allows users to perform the training on a single GPU.

## Important Links

- GitHub: [https://github.com/zjudai/flowertune-llm](https://github.com/zjudai/flowertune-llm)
- HuggingFace: [https://huggingface.co/zjudai/FlowerTune](https://huggingface.co/zjudai/FlowerTune)
- Model Collection: [FlowerTune LoRA Collection](https://huggingface.co/collections/zjudai/flowertune-lora-collection-67ecd5d0dae6145cbf798439)

## Experimental Setup

The dataset partitioning follows the default settings in FlowerTune llm Leaderboard, for a total of `10` rounds.
All settings are defined in `pyproject.toml`.

## Methodology

This baseline performs federated LLM fine-tuning with [LoRA](https://arxiv.org/abs/2106.09685) using the [🤗PEFT](https://huggingface.co/docs/peft/en/index) library.
The clients' models are aggregated with `FedAvg` strategy.
This provides a baseline performance for the benchmark.

### Example: Qwen2.5-7B-Instruct

For example, with the **Qwen/Qwen2.5-7B-Instruct** model we adopted the following fine-tuning methodology:

- **Precision**: `bf16` for model weights.
- **Quantization**: `4-bit` quantization for reduced memory usage.
- **LoRA Configuration**:
  - Rank (r): `32`
  - Alpha: `64`
- **Training Configuration**:
  - Batch size: `8`
  - Maximum number of steps: `10`
  - Total number of rounds: `10`
  - Fraction fit per round: `0.1`
- **Learning Rate Scheduler**:
  - Maximum LR: `5e-5`
  - Minimum LR: `1e-6`
  - Constant learning rate scheduler over steps
- **Strategy**: `FedAvg`

## Environment and Execution

### Environment Setup

First, create and activate the conda environment:

```shell
conda create -n flwr-tune python=3.9
conda activate flwr-tune
```

Project dependencies are defined in `pyproject.toml`. Install them in an activated Python environment with:

```shell
pip install -e .
```


### Running the Training and Evaluation

We use a wrapper script `run_all_experiments.sh` to handle both training and evaluation processes:

```bash
# Example of running experiments
./run_all_experiments.sh --model Qwen/Qwen2.5-7B-Instruct --task general-nlp
```

The wrapper script sets up the proper environment, including:
- Activating the conda environment
- Setting up proxy configurations if needed
- Executing the main experiment runner script with the provided parameters

The actual experiment workflow is implemented in `run_experiments.py`, which is called by the wrapper script.

### Model Saving

The global PEFT model checkpoints are saved every 5 rounds after aggregation on the server side as default, which can be specified with `train.save-every-round` under [tool.flwr.app.config] entry in `pyproject.toml`.

## Evaluation Results
Please find the checkpoint link by clicking each model's name.

### General NLP
The evaluation was conducted on the MMLU (Massive Multitask Language Understanding) benchmark, which tests knowledge across various domains:

| **Model** | **STEM** | **Social Sciences** | **Humanities** | **Average** | **Comm. Costs** |
|-----------|----------|---------------------|----------------|-------------|-----------------|
| [Qwen/Qwen2.5-7B-Instruct](https://huggingface.co/zjudai/flowertune-general-nlp-lora-qwen2.5-7b-instruct) | 52.52% | 79.27% | 60.32% | 64.04% | 1.50 GB |
| [Qwen/Qwen2.5-1.5B-Instruct](https://huggingface.co/zjudai/flowertune-general-nlp-lora-qwen2.5-1.5b-instruct) | 47.13% | 62.30% | 50.54% | 53.32% | 0.65 GB |
| [mistralai/Mistral-7B-Instruct-v0.3](https://huggingface.co/zjudai/flowertune-general-nlp-lora-mistral-7b-instruct-v0.3) | 29.94% | 54.27% | 44.93% | 43.05% | 2.03 GB |
| [meta-llama/Llama-3.1-8B-Instruct](https://huggingface.co/zjudai/flowertune-general-nlp-lora-llama-3.1-8b-instruct) | 22.87% | 39.55% | 32.05% | 31.49% | 2.03 GB |
| [TinyLlama/TinyLlama-1.1B-Chat-v1.0](https://huggingface.co/zjudai/flowertune-general-nlp-lora-tinyllama-1.1b-chat-v1.0) | 14.18% | 21.61% | 21.91% | 19.23% | 0.67 GB |
| [meta-llama/Llama-3.2-1B-Instruct](https://huggingface.co/zjudai/flowertune-general-nlp-lora-llama-3.2-1b-instruct) | 12.88% | 17.61% | 6.16% | 12.22% | 0.51 GB |

### Medical

| **Model** | **PubMedQA** | **MedMCQA** | **MedQA** | **CareQA** |**Average** | **Comm. Costs** |
|-----------|----------|---------------------|----------------|-------------|-----------------|-----------------|
| [meta-llama/Llama-3.1-8B-Instruct](https://huggingface.co/zjudai/flowertune-medical-lora-llama-3.1-8b-instruct) | 59.94% | 69.40% | 59.94% | 57.74% | 61.75% | 2.03 GB |
| [Qwen/Qwen2.5-7B-Instruct](https://huggingface.co/zjudai/flowertune-medical-lora-qwen2.5-7b-instruct) | 65.15% | 56.80% | 65.15% | 55.46% | 60.64% | 1.50 GB |
| [mistralai/Mistral-7B-Instruct-v0.3](https://huggingface.co/zjudai/flowertune-finance-lora-mistral-7b-instruct-v0.3) | 54.40% | 55.20% | 54.40% | 49.80% | 53.45% | 2.03 GB |

### Finance 

| **Model** | **FPB** | **FIQA** | **TFNS** | **Average** | **Comm. Costs** |
|-----------|----------|---------------------|----------------|-------------|-----------------|
| [mistralai/Mistral-7B-Instruct-v0.3](https://huggingface.co/zjudai/flowertune-finance-lora-mistral-7b-instruct-v0.3) | 58.63% | 67.11% | 58.63% | 61.45% | 2.03 GB |


## Hardware Details

The GPU details are as follows.

| **Component** | **Specification**    |
|---------------|----------------------|
| **GPU**       | 1 × NVIDIA A100 GPU with 40 GB Memory |
