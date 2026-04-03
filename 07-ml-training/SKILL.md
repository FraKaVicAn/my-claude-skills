---
name: ml-training
description: Auto-activating ML training skill — PyTorch/TensorFlow/sklearn pipelines, hyperparameter tuning, MLflow/W&B experiment tracking, distributed training, and model evaluation
origin: jeremylongshore/claude-code-plugins-plus-skills/07-ml-training
tools: Read, Write, Edit, Bash, Grep
---

# ML Training Skill

Automated assistance for machine learning model development: data pipelines, training loops, hyperparameter optimization, experiment tracking, and evaluation.

## When to Activate

Activates automatically when you mention: PyTorch, TensorFlow, sklearn, model training, dataset, hyperparameter, MLflow, W&B, wandb, TensorBoard, cross-validation, early stopping, learning rate, gradient, distributed training, feature engineering, confusion matrix.

## Covered Skills (25 total)

| Skill | What it does |
|-------|-------------|
| `pytorch-model-trainer` | Training loops, loss, optimizer, gradient clipping |
| `tensorflow-model-trainer` | Keras model compilation, callbacks, fit() patterns |
| `sklearn-pipeline-builder` | Pipeline with preprocessing + estimator + CV |
| `dataset-loader-creator` | DataLoader, Dataset class, batching, workers |
| `data-augmentation-pipeline` | torchvision/albumentations augmentation chains |
| `hyperparameter-tuner` | Grid search, random search, Bayesian optimization |
| `optuna-study-creator` | Optuna TPE sampler, pruning, dashboard |
| `learning-rate-scheduler` | CosineAnnealing, ReduceLROnPlateau, warmup |
| `model-checkpoint-manager` | Save/load best model, resume training |
| `mlflow-tracking-setup` | Experiment tracking, artifact logging, model registry |
| `wandb-experiment-logger` | W&B run initialization, logging, sweeps |
| `tensorboard-visualizer` | TensorBoard logging, histograms, embeddings |
| `model-evaluation-metrics` | Accuracy, F1, AUC, precision/recall curves |
| `confusion-matrix-generator` | Confusion matrix with seaborn visualization |
| `feature-importance-analyzer` | SHAP values, permutation importance |
| `early-stopping-callback` | Patience-based stopping with best-model restore |
| `mixed-precision-trainer` | AMP with GradScaler for faster GPU training |
| `distributed-training-setup` | DDP with torch.distributed |
| `cross-validation-setup` | StratifiedKFold, time series split |

## PyTorch Training Loop

```python
import torch
import torch.nn as nn
from torch.optim.lr_scheduler import CosineAnnealingLR

def train_epoch(model, loader, optimizer, criterion, device, scaler=None):
    model.train()
    total_loss = 0.0
    for batch_idx, (data, target) in enumerate(loader):
        data, target = data.to(device), target.to(device)
        optimizer.zero_grad()

        if scaler:  # Mixed precision
            with torch.cuda.amp.autocast():
                output = model(data)
                loss = criterion(output, target)
            scaler.scale(loss).backward()
            scaler.unscale_(optimizer)
            torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
            scaler.step(optimizer)
            scaler.update()
        else:
            output = model(data)
            loss = criterion(output, target)
            loss.backward()
            torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
            optimizer.step()

        total_loss += loss.item()
    return total_loss / len(loader)

# Training setup
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model = MyModel().to(device)
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3, weight_decay=1e-4)
scheduler = CosineAnnealingLR(optimizer, T_max=num_epochs)
scaler = torch.cuda.amp.GradScaler() if device.type == "cuda" else None
```

## Optuna Hyperparameter Search

```python
import optuna

def objective(trial):
    lr = trial.suggest_float("lr", 1e-5, 1e-2, log=True)
    dropout = trial.suggest_float("dropout", 0.1, 0.5)
    hidden_size = trial.suggest_categorical("hidden_size", [128, 256, 512])

    model = MyModel(hidden_size=hidden_size, dropout=dropout).to(device)
    optimizer = torch.optim.Adam(model.parameters(), lr=lr)

    val_loss = train_and_evaluate(model, optimizer, n_epochs=10)

    trial.report(val_loss, step=10)
    if trial.should_prune():
        raise optuna.exceptions.TrialPruned()
    return val_loss

study = optuna.create_study(
    direction="minimize",
    pruner=optuna.pruners.MedianPruner(n_warmup_steps=5)
)
study.optimize(objective, n_trials=100, n_jobs=4)
print(f"Best params: {study.best_params}")
```

## MLflow Tracking

```python
import mlflow

with mlflow.start_run(run_name="experiment-v1"):
    mlflow.log_params({"lr": 1e-3, "epochs": 50, "batch_size": 32})
    for epoch in range(num_epochs):
        train_loss = train_epoch(...)
        val_loss, val_acc = evaluate(...)
        mlflow.log_metrics({"train_loss": train_loss, "val_loss": val_loss, "val_acc": val_acc}, step=epoch)
    mlflow.pytorch.log_model(model, "model")
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `07-ml-training` (25 skills)
