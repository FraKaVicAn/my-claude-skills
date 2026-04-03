---
name: aws-skills
description: Auto-activating AWS skill — Lambda, ECS, DynamoDB, S3, VPC, CloudFormation/CDK/SAM, IAM, CloudWatch, SNS/SQS, EventBridge, EKS, and cost optimization
origin: jeremylongshore/claude-code-plugins-plus-skills/13-aws-skills
tools: Read, Write, Edit, Bash, Grep
---

# AWS Skills Skill

Automated assistance for AWS infrastructure: serverless (Lambda/SAM), containers (ECS/EKS), databases (DynamoDB/RDS), networking (VPC), IaC (CloudFormation/CDK), and operations (CloudWatch/SNS/SQS).

## When to Activate

Activates automatically when you mention: Lambda, ECS, DynamoDB, S3, VPC, CloudFormation, CDK, SAM, IAM, CloudWatch, SNS, SQS, EventBridge, EKS, API Gateway, RDS, ElastiCache, Route53, CloudFront, Step Functions, cost optimization.

## Covered Skills (25 total)

| Skill | What it does |
|-------|-------------|
| `lambda-function-generator` | Lambda handler, event parsing, error handling |
| `lambda-layer-creator` | Shared dependencies as Lambda layers |
| `ecs-task-definition-creator` | ECS Fargate task definitions |
| `step-functions-workflow` | Step Functions state machine definition |
| `dynamodb-table-designer` | DynamoDB table with GSIs, LSIs, capacity |
| `s3-bucket-policy-generator` | S3 bucket policies and ACLs |
| `s3-lifecycle-config` | S3 lifecycle rules for cost optimization |
| `vpc-network-designer` | VPC with public/private subnets, NAT |
| `security-group-generator` | Security group rules with least privilege |
| `route53-record-manager` | DNS records, health checks, routing policies |
| `cloudfront-distribution-setup` | CloudFront with OAC, caching behaviors |
| `api-gateway-config` | REST/HTTP API Gateway with auth |
| `cloudformation-template-creator` | CloudFormation YAML templates |
| `cdk-stack-generator` | AWS CDK TypeScript/Python stacks |
| `sam-template-builder` | SAM template for serverless apps |
| `iam-policy-creator` | Least-privilege IAM policies |
| `cloudwatch-alarm-creator` | CloudWatch alarms with SNS notifications |
| `sns-topic-config` | SNS topics with subscriptions |
| `sqs-queue-setup` | SQS queues with DLQ configuration |
| `eventbridge-rule-creator` | EventBridge rules and targets |
| `cost-optimization-analyzer` | Cost analysis and right-sizing recommendations |
| `eks-cluster-config` | EKS cluster with managed node groups |

## Lambda Function Pattern

```python
import json
import boto3
from aws_lambda_powertools import Logger, Tracer, Metrics
from aws_lambda_powertools.utilities.typing import LambdaContext
from aws_lambda_powertools.utilities.data_classes import APIGatewayProxyEvent

logger = Logger()
tracer = Tracer()
metrics = Metrics()

dynamodb = boto3.resource("dynamodb")
table = dynamodb.Table("MyTable")

@logger.inject_lambda_context(log_event=True)
@tracer.capture_lambda_handler
@metrics.log_metrics(capture_cold_start_metric=True)
def handler(event: dict, context: LambdaContext) -> dict:
    try:
        api_event = APIGatewayProxyEvent(event)
        item_id = api_event.path_parameters["id"]

        item = table.get_item(Key={"id": item_id}).get("Item")
        if not item:
            return {"statusCode": 404, "body": json.dumps({"error": "Not found"})}

        logger.info("Item retrieved", item_id=item_id)
        return {"statusCode": 200, "body": json.dumps(item), "headers": {"Content-Type": "application/json"}}

    except Exception as e:
        logger.exception("Unexpected error")
        return {"statusCode": 500, "body": json.dumps({"error": "Internal server error"})}
```

## CDK Stack (TypeScript)

```typescript
import * as cdk from 'aws-cdk-lib'
import * as lambda from 'aws-cdk-lib/aws-lambda'
import * as apigateway from 'aws-cdk-lib/aws-apigateway'
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb'

export class ApiStack extends cdk.Stack {
  constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
    super(scope, id, props)

    const table = new dynamodb.Table(this, 'ItemsTable', {
      partitionKey: { name: 'id', type: dynamodb.AttributeType.STRING },
      billingMode: dynamodb.BillingMode.PAY_PER_REQUEST,
      pointInTimeRecovery: true,
      encryption: dynamodb.TableEncryption.AWS_MANAGED,
    })

    const fn = new lambda.Function(this, 'ApiHandler', {
      runtime: lambda.Runtime.PYTHON_3_12,
      handler: 'handler.handler',
      code: lambda.Code.fromAsset('src'),
      environment: { TABLE_NAME: table.tableName },
      tracing: lambda.Tracing.ACTIVE,
      timeout: cdk.Duration.seconds(30),
    })

    table.grantReadWriteData(fn)
    new apigateway.LambdaRestApi(this, 'Api', { handler: fn })
  }
}
```

## Links

- [Repository](https://github.com/jeremylongshore/claude-code-plugins-plus-skills)
- Category: `13-aws-skills` (25 skills)
