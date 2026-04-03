---
name: gcp-skills
description: Auto-activating GCP skill — Cloud Functions, Cloud Run, GKE, BigQuery, Vertex AI, Pub/Sub, Cloud SQL, Firestore, GCS, IAM, Firebase, and Cloud Monitoring
origin: jeremylongshore/claude-code-plugins-plus-skills/14-gcp-skills
tools: Read, Write, Edit, Bash, Grep
---

# GCP Skills Skill

Automated assistance for Google Cloud Platform: serverless, containers, BigQuery analytics, Vertex AI, Pub/Sub messaging, storage, security, and monitoring.

## When to Activate

Activates automatically when you mention: Cloud Function, Cloud Run, GKE, BigQuery, Vertex AI, Pub/Sub, Cloud SQL, Firestore, GCS, Cloud Storage, IAM, Firebase, Cloud Monitoring, Cloud Logging, Cloud Tasks, Cloud Scheduler, VPC, service account.

## Covered Skills (25 total)

| Skill | What it does |
|-------|-------------|
| `cloud-function-generator` | Cloud Functions (gen2) with trigger config |
| `cloud-run-service-config` | Cloud Run service YAML with scaling |
| `gke-cluster-config` | GKE Autopilot / Standard cluster setup |
| `bigquery-table-creator` | BigQuery table schema, partitioning, clustering |
| `bigquery-view-generator` | Authorized views, materialized views |
| `bigquery-scheduled-query` | Scheduled queries for ETL |
| `bigquery-ml-model-creator` | BQML model training and evaluation |
| `vertex-ai-pipeline-creator` | Vertex AI Pipelines with Kubeflow components |
| `vertex-ai-endpoint-config` | Model deployment to Vertex AI endpoints |
| `pubsub-topic-setup` | Pub/Sub topic with schema |
| `pubsub-subscription-config` | Push/pull subscriptions with DLQ |
| `cloud-tasks-queue-setup` | Cloud Tasks queue with HTTP targets |
| `cloud-scheduler-job-creator` | Cron jobs with Cloud Scheduler |
| `cloud-sql-instance-setup` | Cloud SQL PostgreSQL/MySQL instance |
| `firestore-index-creator` | Firestore composite indexes |
| `gcs-bucket-config` | GCS bucket with lifecycle, versioning |
| `iam-binding-creator` | IAM bindings with condition expressions |
| `service-account-manager` | Service account creation and key management |
| `vpc-network-setup` | VPC with subnets, firewall rules |
| `cloud-monitoring-alert` | Alerting policies and notification channels |
| `firebase-rules-generator` | Firestore/Storage security rules |

## Cloud Run Service

```yaml
# service.yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: my-service
  annotations:
    run.googleapis.com/ingress: all
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/minScale: "1"
        autoscaling.knative.dev/maxScale: "100"
        run.googleapis.com/cpu-throttling: "false"
    spec:
      serviceAccountName: my-sa@project.iam.gserviceaccount.com
      containers:
      - image: gcr.io/project/my-service:latest
        resources:
          limits: { cpu: "2", memory: "1Gi" }
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef: { name: db-url, key: latest }
        ports:
        - containerPort: 8080
```

```bash
gcloud run services replace service.yaml --region=us-central1
```

## BigQuery Partitioned Table

```python
from google.cloud import bigquery

client = bigquery.Client()
schema = [
    bigquery.SchemaField("event_id", "STRING", mode="REQUIRED"),
    bigquery.SchemaField("user_id", "STRING"),
    bigquery.SchemaField("event_type", "STRING"),
    bigquery.SchemaField("timestamp", "TIMESTAMP"),
    bigquery.SchemaField("properties", "JSON"),
]
table = bigquery.Table(f"project.dataset.events", schema=schema)
table.time_partitioning = bigquery.TimePartitioning(field="timestamp", type_="DAY", expiration_ms=7776000000)
table.clustering_fields = ["event_type", "user_id"]
table = client.create_table(table, exists_ok=True)
```

## Pub/Sub Publisher

```python
from google.cloud import pubsub_v1
import json

publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path("my-project", "my-topic")

def publish_event(event: dict) -> str:
    data = json.dumps(event).encode("utf-8")
    future = publisher.publish(
        topic_path, data,
        event_type=event.get("type", "unknown"),  # message attributes
        origin="my-service"
    )
    return future.result()  # message ID
```

## Firestore Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if request.auth != null && request.auth.uid == userId
                   && request.resource.data.keys().hasOnly(['name', 'email', 'updatedAt']);
    }
    match /posts/{postId} {
      allow read: if resource.data.published == true || request.auth.uid == resource.data.authorId;
      allow create: if request.auth != null;
      allow update, delete: if request.auth.uid == resource.data.authorId;
    }
  }
}
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `14-gcp-skills` (25 skills)
