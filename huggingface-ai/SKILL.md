---
name: huggingface-ai
description: Hugging Face Hub — model training (TRL/SFT/DPO), evaluation, datasets, inference, HF CLI, Replicate API, and fal.ai image generation. Auto-activates for ML model or HuggingFace tasks.
origin: VoltAgent/awesome-agent-skills (huggingface, replicate, fal.ai)
tools: Read, Write, Edit, Bash
---

# Hugging Face / AI Models Skills

Official skills from Hugging Face, Replicate, and fal.ai teams.

## Hugging Face CLI

```bash
# Install
pip install huggingface_hub

# Login
huggingface-cli login

# Download model
huggingface-cli download meta-llama/Llama-3.1-8B-Instruct

# Upload model
huggingface-cli upload my-org/my-model ./my-model-dir

# Download dataset
huggingface-cli download datasets/my-dataset --repo-type dataset

# List files in repo
huggingface-cli list my-org/my-model
```

## Inference (transformers)

```python
from transformers import pipeline, AutoTokenizer, AutoModelForCausalLM
import torch

# Easy pipeline API
generator = pipeline('text-generation', model='meta-llama/Llama-3.2-1B')
result = generator("Explain transformers in simple terms:", max_new_tokens=200)
print(result[0]['generated_text'])

# Direct model loading
model_name = "microsoft/phi-3-mini-4k-instruct"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    torch_dtype=torch.bfloat16,
    device_map="auto",
)

messages = [{"role": "user", "content": "What is machine learning?"}]
text = tokenizer.apply_chat_template(messages, tokenize=False, add_generation_prompt=True)
inputs = tokenizer(text, return_tensors='pt').to(model.device)
outputs = model.generate(**inputs, max_new_tokens=500)
print(tokenizer.decode(outputs[0][inputs['input_ids'].shape[1]:], skip_special_tokens=True))
```

## Inference API (serverless)

```python
from huggingface_hub import InferenceClient

client = InferenceClient(api_key=os.environ['HF_TOKEN'])

# Text generation
response = client.text_generation(
    "Summarize this article: ...",
    model="meta-llama/Llama-3.1-8B-Instruct",
    max_new_tokens=500,
)

# Chat completion (OpenAI-compatible)
response = client.chat.completions.create(
    model="meta-llama/Llama-3.1-8B-Instruct",
    messages=[{"role": "user", "content": "Hello!"}],
    max_tokens=500,
)

# Image generation
image = client.text_to_image(
    "A photo of an astronaut on the moon",
    model="black-forest-labs/FLUX.1-schnell",
)
image.save("output.png")
```

## Fine-tuning with TRL

```python
from trl import SFTTrainer, SFTConfig
from datasets import load_dataset
from transformers import AutoModelForCausalLM, AutoTokenizer

# Load base model
model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-3.2-1B")
tokenizer = AutoTokenizer.from_pretrained("meta-llama/Llama-3.2-1B")

# Load dataset
dataset = load_dataset("json", data_files="train.jsonl", split="train")

# SFT Training
trainer = SFTTrainer(
    model=model,
    tokenizer=tokenizer,
    train_dataset=dataset,
    args=SFTConfig(
        output_dir="./output",
        num_train_epochs=3,
        per_device_train_batch_size=4,
        gradient_accumulation_steps=4,
        learning_rate=2e-4,
        bf16=True,
        logging_steps=10,
        save_steps=100,
    ),
)
trainer.train()
trainer.save_model("./my-finetuned-model")
```

### DPO (Preference Training)

```python
from trl import DPOTrainer, DPOConfig

trainer = DPOTrainer(
    model=model,
    ref_model=ref_model,
    tokenizer=tokenizer,
    train_dataset=dataset,  # must have 'prompt', 'chosen', 'rejected' columns
    args=DPOConfig(
        output_dir="./dpo-output",
        beta=0.1,           # KL penalty coefficient
        num_train_epochs=1,
    ),
)
trainer.train()
```

### GGUF Conversion (for llama.cpp/Ollama)

```bash
pip install llama-cpp-python

# Convert to GGUF
python llama.cpp/convert-hf-to-gguf.py ./my-model --outfile ./my-model.gguf

# Quantize
./llama.cpp/quantize ./my-model.gguf ./my-model-Q4_K_M.gguf Q4_K_M

# Upload to Hub
huggingface-cli upload my-org/my-model-gguf ./my-model-Q4_K_M.gguf
```

## Datasets

```python
from datasets import load_dataset, Dataset, DatasetDict
import pandas as pd

# Load from Hub
dataset = load_dataset("imdb", split="train")
dataset = load_dataset("csv", data_files="data.csv")
dataset = load_dataset("json", data_files={"train": "train.jsonl", "test": "test.jsonl"})

# Create from pandas
df = pd.read_csv("data.csv")
dataset = Dataset.from_pandas(df)

# Transform
dataset = dataset.map(lambda x: {"text": x["prompt"] + x["response"]})
dataset = dataset.filter(lambda x: len(x["text"]) > 100)
dataset = dataset.train_test_split(test_size=0.1)

# Push to Hub
dataset.push_to_hub("my-org/my-dataset")
```

## Evaluation

```bash
pip install lighteval

# Evaluate model on benchmarks
lighteval accelerate \
  --model_args "pretrained=my-org/my-model" \
  --tasks "leaderboard|hellaswag|0|1,leaderboard|arc:challenge|0|1" \
  --output_dir ./eval-results
```

## HF Jobs (Remote Compute)

```python
from huggingface_hub import HfApi

api = HfApi()

# Run training job on HF infrastructure
job = api.run_as_job(
    script="train.py",
    args=["--model", "llama-3.2-1b", "--epochs", "3"],
    hardware="a10g-small",
    timeout=3600,
)
print(f"Job ID: {job.id}, Status: {job.status}")
```

## Replicate — Run Any AI Model

```python
import replicate

# Run image generation
output = replicate.run(
    "black-forest-labs/flux-1.1-pro",
    input={
        "prompt": "A futuristic city at sunset",
        "aspect_ratio": "16:9",
        "output_format": "webp",
    }
)
print(output)  # URL to generated image

# Run video generation
output = replicate.run(
    "minimax/video-01",
    input={"prompt": "A cat playing piano", "duration": 5}
)

# Stream output
for event in replicate.stream("meta/llama-3.1-405b-instruct", input={"prompt": "Hello"}):
    print(str(event), end="", flush=True)
```

## fal.ai — Fast AI Inference

```python
import fal_client

# Image generation (FLUX)
result = fal_client.run(
    "fal-ai/flux/schnell",
    arguments={
        "prompt": "A robot painting a landscape",
        "image_size": "landscape_16_9",
        "num_images": 1,
    }
)
print(result["images"][0]["url"])

# Async for long tasks
handle = fal_client.submit("fal-ai/flux-pro", arguments={"prompt": "..."})
result = fal_client.result("fal-ai/flux-pro", handle.request_id)

# Real-time with queue
@fal_client.on_queue_update
def update(update):
    print(f"Progress: {update.status}")

result = fal_client.subscribe("fal-ai/sadtalker", arguments={...}, on_queue_update=update)
```

## Sources
- huggingface/hf-cli
- huggingface/hugging-face-datasets
- huggingface/hugging-face-model-trainer
- huggingface/hugging-face-evaluation
- huggingface/hugging-face-jobs
- huggingface/hugging-face-dataset-viewer
- replicate/replicate
- fal.ai team skills
