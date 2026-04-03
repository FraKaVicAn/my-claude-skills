---
name: ml-deployment
description: Auto-activating ML deployment skill — FastAPI/Flask/TorchServe serving, ONNX export, SageMaker/Vertex AI/Azure ML deployers, model quantization, A/B testing, and drift detection
origin: jeremylongshore/claude-code-plugins-plus-skills/08-ml-deployment
tools: Read, Write, Edit, Bash, Grep
---

# ML Deployment Skill

Automated assistance for deploying ML models to production: API serving, model export, cloud deployment, optimization, monitoring, and A/B testing.

## When to Activate

Activates automatically when you mention: model deployment, serving, inference, ONNX, TorchScript, SageMaker, Vertex AI, Azure ML, Triton, quantization, model pruning, model drift, A/B test, canary deployment, feature store, batch inference, GPU optimization.

## Covered Skills (26 total)

| Skill | What it does |
|-------|-------------|
| `fastapi-ml-endpoint` | FastAPI inference endpoint with batching |
| `flask-ml-api-creator` | Flask model serving with health checks |
| `tensorflow-serving-setup` | TF Serving config, REST/gRPC endpoints |
| `torchserve-config-generator` | TorchServe model archive and config |
| `model-export-helper` | Export to ONNX, TorchScript, SavedModel |
| `onnx-converter` | PyTorch/TF → ONNX with validation |
| `sagemaker-endpoint-deployer` | SageMaker real-time endpoint setup |
| `vertex-ai-deployer` | Vertex AI model registry and endpoint |
| `azure-ml-deployer` | Azure ML managed online endpoints |
| `triton-inference-config` | Triton server model repository config |
| `model-quantization-tool` | INT8/FP16 quantization with accuracy check |
| `model-pruning-helper` | Structured/unstructured pruning patterns |
| `model-versioning-manager` | Semantic versioning for ML models |
| `model-drift-detector` | Data drift and concept drift monitoring |
| `batch-inference-pipeline` | Offline batch scoring pipelines |
| `a-b-test-config-creator` | Traffic splitting for model comparison |
| `canary-deployment-setup` | Gradual rollout with monitoring |
| `prediction-monitor` | Prediction logging and alerting |
| `gpu-resource-optimizer` | GPU memory, batching, and throughput tuning |

## FastAPI Model Server

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import torch
import numpy as np
from typing import List
import asyncio

app = FastAPI()
model = None  # loaded at startup

@app.on_event("startup")
async def load_model():
    global model
    model = torch.load("model.pt", weights_only=True)
    model.eval()

class PredictRequest(BaseModel):
    inputs: List[List[float]]
    threshold: float = 0.5

class PredictResponse(BaseModel):
    predictions: List[float]
    labels: List[int]

@app.post("/predict", response_model=PredictResponse)
async def predict(request: PredictRequest):
    if model is None:
        raise HTTPException(503, "Model not loaded")
    with torch.no_grad():
        inputs = torch.tensor(request.inputs, dtype=torch.float32)
        outputs = model(inputs).sigmoid().numpy()
    return PredictResponse(
        predictions=outputs.tolist(),
        labels=(outputs >= request.threshold).astype(int).tolist()
    )

@app.get("/health")
async def health():
    return {"status": "healthy", "model_loaded": model is not None}
```

## ONNX Export

```python
import torch
import onnx
import onnxruntime as ort

def export_to_onnx(model, sample_input, output_path: str):
    model.eval()
    torch.onnx.export(
        model, sample_input, output_path,
        export_params=True,
        opset_version=17,
        input_names=["input"],
        output_names=["output"],
        dynamic_axes={"input": {0: "batch_size"}, "output": {0: "batch_size"}}
    )
    # Validate
    onnx_model = onnx.load(output_path)
    onnx.checker.check_model(onnx_model)
    # Benchmark
    session = ort.InferenceSession(output_path, providers=["CUDAExecutionProvider", "CPUExecutionProvider"])
    result = session.run(None, {"input": sample_input.numpy()})
    return result
```

## Model Drift Detection

```python
from scipy import stats

def detect_data_drift(reference_data, current_data, threshold: float = 0.05) -> dict:
    """Kolmogorov-Smirnov test for each feature."""
    drift_report = {}
    for feature_idx in range(reference_data.shape[1]):
        stat, p_value = stats.ks_2samp(
            reference_data[:, feature_idx],
            current_data[:, feature_idx]
        )
        drift_report[f"feature_{feature_idx}"] = {
            "ks_statistic": stat,
            "p_value": p_value,
            "drift_detected": p_value < threshold
        }
    return drift_report
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `08-ml-deployment` (26 skills)
