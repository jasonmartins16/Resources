# AWS: Interview Study Guide

All 36 AWS topics from the study list. Each entry explains what the service or concept is, when to use it, and the points interviewers commonly probe.

> **Cost warning: some of these services cost money.** AWS bills for most resources while they exist or run, not only while you use them, and the free tier is limited in amount and duration. The commands in this guide create real resources, so run them in a sandbox account and delete everything when you finish practicing. Charges are especially easy to miss for these:
>
> - **Compute and databases:** running EC2 instances, RDS and Aurora databases, Redshift clusters, and Fargate tasks keep billing until stopped or deleted.
> - **Networking:** NAT Gateways (hourly plus per-GB), load balancers, unattached Elastic IPs, VPN connections, Transit Gateway attachments and Direct Connect, plus data transfer out to the internet.
> - **Containers and ML:** the EKS control plane (a flat hourly fee, on top of the worker nodes), SageMaker endpoints and notebooks, and OpenSearch domains.
> - **Streaming and security:** Kinesis shards (hourly), WAF web ACLs, GuardDuty, Config rules, extra CloudTrail trails, Secrets Manager secrets and KMS keys.
> - **Storage left behind:** EBS volumes and snapshots, and S3 objects kept after you are done.
>
> Before you start: create a Budget alert (see item 36), tag what you create, and check the Billing console for anything still running. Prices differ by region and change over time, so confirm on the official AWS pricing pages.

## Part 1: Core Compute, Storage and Networking

### 1. Amazon EC2

Resizable virtual servers (instances) in the cloud. You choose the instance type (CPU/memory/network profile: general purpose, compute, memory or GPU optimized), an AMI (OS image), storage and networking. Pricing models: On-Demand, Reserved/Savings Plans, Spot (up to \~90% cheaper but interruptible). Use for full control over the OS or workloads that do not fit serverless.

**Use case:** host a web application or API on a server you control.

**How to use:** choose an AMI (OS image), an instance type and a security group that allows the ports you need, then launch and connect.

```bash
aws ec2 run-instances --image-id ami-0abcdef1234567890 --instance-type t3.micro \
  --key-name my-key --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 --count 1

ssh -i my-key.pem ec2-user@<public-ip>
```

IDs above are placeholders; the console launch wizard does the same thing.

### 2. Amazon S3

Object storage with virtually unlimited capacity and 11 nines of durability. Data lives in **buckets** as **objects** (key, data, metadata). Features: versioning, encryption, lifecycle rules, replication, pre-signed URLs, static website hosting, event notifications. It is not a filesystem or a block device. Strong read-after-write consistency.

**Use case:** store user uploads (photos, documents) and share them securely.

**How to use:** create a bucket, upload objects, and hand out time-limited links instead of making the bucket public.

```bash
aws s3 mb s3://my-app-uploads-12345
aws s3 cp photo.jpg s3://my-app-uploads-12345/images/photo.jpg
aws s3 ls s3://my-app-uploads-12345/images/
aws s3 presign s3://my-app-uploads-12345/images/photo.jpg --expires-in 3600
```

Bucket names are globally unique, so pick your own.

### 3. AWS IAM

Controls **who** can do **what** on **which** resources. Building blocks: users, groups, **roles** (temporary credentials assumed by services or people), and **policies** (JSON documents with Effect, Action, Resource, Condition). Principles: least privilege, explicit deny overrides allow, prefer roles over long-lived access keys, enable MFA, and never use the root account for daily work.

**Use case:** let an application on EC2 read one S3 bucket without storing access keys.

**How to use:** create a policy that allows only what is needed, attach it to an IAM role, and assign that role to the instance (as an instance profile).

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::my-app-uploads-12345/*"
  }]
}
```

The app then uses the SDK with no credentials in code; AWS supplies temporary ones from the role.

### 4. Amazon VPC

Your logically isolated private network. Components: CIDR range, **subnets** (public has a route to an Internet Gateway; private does not), route tables, Internet Gateway, **NAT Gateway** (outbound-only internet for private subnets), VPC endpoints (private access to AWS services), peering and Transit Gateway. Typical layout: load balancer in public subnets, app and database in private subnets across multiple AZs.

**Use case:** a three-tier web app with public load balancers and private app and database servers.

**How to use:** create the VPC, carve out subnets in two AZs, attach an internet gateway to the public ones, and route private subnets through a NAT gateway.

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16
aws ec2 create-subnet --vpc-id vpc-0123 --cidr-block 10.0.1.0/24 --availability-zone us-east-1a
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id vpc-0123 --internet-gateway-id igw-0123
```

A subnet is public only if its route table sends 0.0.0.0/0 to the internet gateway.

### 5. Amazon EBS

Persistent **block storage** volumes attached to a single EC2 instance (within the same AZ), like a virtual hard drive. Types: gp3/gp2 (general SSD), io2 (high IOPS), st1/sc1 (throughput/cold HDD). Supports snapshots (stored in S3, incremental) and encryption. Data survives instance stop; by default the root volume is deleted on termination.

**Use case:** a dedicated data disk for a database running on EC2.

**How to use:** create a volume in the same AZ as the instance, attach it, then format and mount it on the server.

```bash
aws ec2 create-volume --size 100 --volume-type gp3 --availability-zone us-east-1a
aws ec2 attach-volume --volume-id vol-0123 --instance-id i-0123 --device /dev/sdf

# on the instance (it may appear as /dev/nvme1n1)
sudo mkfs -t xfs /dev/sdf
sudo mkdir /data && sudo mount /dev/sdf /data
```

### 6. Elastic Load Balancing

Spreads traffic across targets in multiple AZs with health checks. **ALB**: Layer 7, path/host-based routing, great for HTTP and microservices. **NLB**: Layer 4, ultra-low latency, static IPs, millions of requests per second. **GWLB**: for network appliances. Classic LB is legacy.

**Use case:** spread web traffic across two instances and stop sending requests to an unhealthy one.

**How to use:** create an Application Load Balancer, a target group with a health check, register the instances, and add a listener.

```bash
aws elbv2 create-load-balancer --name web-alb --subnets subnet-aaa subnet-bbb --security-groups sg-0123
aws elbv2 create-target-group --name web-tg --protocol HTTP --port 80 --vpc-id vpc-0123 --health-check-path /health
aws elbv2 register-targets --target-group-arn <tg-arn> --targets Id=i-aaa Id=i-bbb
aws elbv2 create-listener --load-balancer-arn <alb-arn> --protocol HTTP --port 80 \
  --default-actions Type=forward,TargetGroupArn=<tg-arn>
```

### 7. EC2 Auto Scaling

Automatically adds or removes instances in an Auto Scaling Group using a launch template, min/desired/max sizes and scaling policies: target tracking (e.g. keep CPU at 50%), step, scheduled and predictive. Replaces unhealthy instances for self-healing and works hand in hand with a load balancer.

**Use case:** keep a web fleet at about 50% CPU, adding servers at peak and removing them when quiet.

**How to use:** create an Auto Scaling group from a launch template, then attach a target-tracking policy.

```bash
aws autoscaling create-auto-scaling-group --auto-scaling-group-name web-asg \
  --launch-template LaunchTemplateName=web-lt,Version='$Latest' \
  --min-size 2 --max-size 6 --desired-capacity 2 \
  --vpc-zone-identifier "subnet-aaa,subnet-bbb"

aws autoscaling put-scaling-policy --auto-scaling-group-name web-asg \
  --policy-name cpu50 --policy-type TargetTrackingScaling \
  --target-tracking-configuration file://cpu50.json
```

`cpu50.json`:

```json
{"PredefinedMetricSpecification": {"PredefinedMetricType": "ASGAverageCPUUtilization"}, "TargetValue": 50.0}
```

### 8. Amazon Route 53

Scalable DNS plus domain registration and health checks. Routing policies: simple, weighted, latency-based, failover, geolocation, geoproximity, multi-value. **Alias records** map the zone apex to AWS resources (ELB, CloudFront, S3) and are free of query charges, unlike CNAMEs, which cannot be used at the apex.

**Use case:** point `www.example.com` at your load balancer.

**How to use:** create a hosted zone for your domain, then add an alias record to the load balancer.

```bash
aws route53 change-resource-record-sets --hosted-zone-id Z123EXAMPLE --change-batch file://alias.json
```

`alias.json`:

```json
{"Changes": [{"Action": "UPSERT", "ResourceRecordSet": {
  "Name": "www.example.com", "Type": "A",
  "AliasTarget": {"HostedZoneId": "<ALB hosted zone id>",
    "DNSName": "web-alb-123.us-east-1.elb.amazonaws.com", "EvaluateTargetHealth": true}}}]}
```

## Part 2: Databases and Containers

### 9. Amazon RDS

Managed relational databases (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server). AWS handles patching, backups and failover. **Multi-AZ** gives a synchronous standby for high availability; **read replicas** are asynchronous copies that scale reads. You cannot SSH into the host.

**Use case:** a managed PostgreSQL database for an application, with automatic failover.

**How to use:** create the instance with Multi-AZ on, let AWS manage the admin password in Secrets Manager, then connect with any client.

```bash
aws rds create-db-instance --db-instance-identifier app-db --engine postgres \
  --db-instance-class db.t3.micro --allocated-storage 20 \
  --master-username appadmin --manage-master-user-password --multi-az

psql -h <endpoint> -U appadmin -d postgres
```

### 10. Amazon DynamoDB

Fully managed serverless NoSQL key-value and document database with single-digit-millisecond latency at any scale. Design around access patterns: partition key (and optional sort key), GSIs and LSIs. Capacity modes: on-demand or provisioned. Features: TTL, streams, global tables, DAX cache, transactions. Avoid hot partitions and scans.

**Use case:** shopping carts or user sessions that need millisecond reads at any scale.

**How to use:** choose a partition key that matches how you look data up, then read and write by key.

```bash
aws dynamodb create-table --table-name Carts \
  --attribute-definitions AttributeName=userId,AttributeType=S \
  --key-schema AttributeName=userId,KeyType=HASH --billing-mode PAY_PER_REQUEST

aws dynamodb put-item --table-name Carts \
  --item '{"userId": {"S": "u123"}, "items": {"L": [{"S": "book"}]}}'

aws dynamodb get-item --table-name Carts --key '{"userId": {"S": "u123"}}'
```

### 11. AWS Lambda

Serverless functions that run in response to events (API Gateway, S3, SQS, EventBridge, schedules). You pay per request and per millisecond of duration. Limits: 15-minute timeout, memory up to 10 GB, stateless. Know **cold starts**, concurrency limits (reserved and provisioned), and layers.

**Use case:** run small pieces of backend code on demand, such as an API handler or a file-processing step, with no servers to manage.

**How to use:** write a handler, zip it, create the function with an execution role, and invoke it.

```python
# app.py
def handler(event, context):
    name = event.get("name", "world")
    return {"statusCode": 200, "body": f"Hello, {name}!"}
```

```bash
zip function.zip app.py
aws lambda create-function --function-name hello --runtime python3.12 \
  --handler app.handler --role <role-arn> --zip-file fileb://function.zip
aws lambda invoke --function-name hello --payload '{"name": "Sam"}' \
  --cli-binary-format raw-in-base64-out out.json
```

### 12. Amazon EKS

Managed Kubernetes control plane on AWS. You run worker nodes as managed node groups, self-managed EC2 or Fargate. Choose it when you need Kubernetes portability and ecosystem; choose ECS for a simpler AWS-native option.

**Use case:** run containerized microservices on Kubernetes with AWS managing the control plane.

**How to use:** create a cluster with the `eksctl` tool, then deploy with standard `kubectl` commands.

```bash
eksctl create cluster --name demo --region us-east-1 --nodes 2
kubectl apply -f deployment.yaml
kubectl get pods
```

### 13. Amazon CloudFront

A CDN with a global network of edge locations that caches content close to users, reducing latency and origin load. Origins: S3, ALB, custom. Features: TTLs and cache policies, invalidations, HTTPS, signed URLs/cookies, Origin Access Control to keep S3 private, Lambda@Edge and CloudFront Functions, AWS WAF and Shield integration.

**Use case:** serve a static website from S3 to users worldwide, over HTTPS, with low latency.

**How to use:** create a distribution with your S3 bucket as the origin (using Origin Access Control so the bucket stays private), then invalidate cached files after a deploy.

```bash
aws cloudfront create-invalidation --distribution-id E123EXAMPLE --paths "/index.html" "/css/*"
```

### 14. Amazon CloudWatch

Monitoring and observability: **metrics** (built-in plus custom), **Logs** (log groups and streams, Logs Insights queries), **Alarms** (trigger SNS or Auto Scaling actions), dashboards and Events (now EventBridge). Memory and disk metrics on EC2 require the CloudWatch agent. CloudTrail is different: it records API calls for auditing.

**Use case:** get alerted when a server's CPU stays high, and tail application logs.

**How to use:** create an alarm on a metric and send it to an SNS topic.

```bash
aws cloudwatch put-metric-alarm --alarm-name web-high-cpu \
  --namespace AWS/EC2 --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-0123 --statistic Average \
  --period 300 --evaluation-periods 2 --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:ops-alerts

aws logs tail /aws/lambda/hello --follow
```

### 15. Amazon SQS

Fully managed message queue that decouples components. **Standard** queues give at-least-once delivery and best-effort ordering with nearly unlimited throughput; **FIFO** queues give ordering and exactly-once processing with lower throughput. Key concepts: visibility timeout, long polling, dead-letter queues, and message retention up to 14 days.

**Use case:** accept orders instantly and process them in the background, so traffic spikes do not overwhelm the processor.

**How to use:** producers send messages; workers receive, process, then delete them.

```bash
aws sqs create-queue --queue-name orders
aws sqs send-message --queue-url <queue-url> --message-body '{"orderId": 42}'
aws sqs receive-message --queue-url <queue-url> --wait-time-seconds 10
aws sqs delete-message --queue-url <queue-url> --receipt-handle <handle>
```

If a worker does not delete the message in time, it becomes visible again and is retried.

### 16. Amazon SNS

Pub/sub notifications: a publisher sends to a **topic** and SNS fans out to many subscribers (SQS, Lambda, HTTP/S, email, SMS, mobile push). The common **fan-out pattern** is SNS topic to multiple SQS queues. Supports message filtering and FIFO topics.

**Use case:** when an order is placed, notify the operations team and also feed other systems.

**How to use:** create a topic, subscribe endpoints (email, SQS, Lambda), then publish once.

```bash
aws sns create-topic --name order-events
aws sns subscribe --topic-arn <topic-arn> --protocol email --notification-endpoint ops@example.com
aws sns publish --topic-arn <topic-arn> --message "Order 42 placed"
```

The email subscriber must confirm the subscription from the message AWS sends.

### 17. Amazon API Gateway

Managed front door for APIs: REST, HTTP and WebSocket APIs. Provides routing, authentication (IAM, Cognito, Lambda authorizers), throttling, caching, request validation, usage plans and stages. HTTP APIs are cheaper and faster; REST APIs have more features. Classic pairing: API Gateway plus Lambda.

**Use case:** expose a Lambda function as a public HTTPS API.

**How to use:** quick-create an HTTP API with the function as its target, allow API Gateway to invoke it, then call the endpoint URL it returns.

```bash
aws apigatewayv2 create-api --name hello-api --protocol-type HTTP \
  --target arn:aws:lambda:us-east-1:123456789012:function:hello

curl https://<api-id>.execute-api.us-east-1.amazonaws.com/
```

### 18. Amazon ECS and AWS Fargate

**ECS** is AWS's container orchestrator (clusters, task definitions, services). **Fargate** is the serverless compute engine for containers: no servers to manage, you size CPU/memory per task and pay for what you use. The alternative launch type is ECS on EC2 for more control and potentially lower cost.

**Use case:** run a containerized API on Fargate with no servers to patch.

**How to use:** push an image to ECR, describe it in a task definition (image, CPU, memory, ports), then run it as a service.

```bash
aws ecs create-cluster --cluster-name demo
aws ecs register-task-definition --cli-input-json file://taskdef.json
aws ecs create-service --cluster demo --service-name web --task-definition web:1 \
  --desired-count 2 --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-aaa,subnet-bbb],securityGroups=[sg-0123],assignPublicIp=ENABLED}"
```

### 19. AWS CloudFormation (and Terraform as an alternative)

Infrastructure as Code: declare resources in YAML/JSON templates; CloudFormation creates **stacks**, supports change sets, drift detection and rollback. **Terraform** is the multi-cloud alternative using HCL with a state file and a plan/apply workflow. IaC gives repeatability, version control and review for infrastructure.

**Use case:** create the same infrastructure repeatedly (dev, test, prod) from a reviewed file instead of clicking in the console.

**How to use:** describe resources in a template, deploy it as a stack, and delete the stack to remove everything it created.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  AppBucket:
    Type: AWS::S3::Bucket
    Properties:
      VersioningConfiguration:
        Status: Enabled
```

```bash
aws cloudformation deploy --template-file template.yaml --stack-name app-storage
aws cloudformation delete-stack --stack-name app-storage
```

### 20. Amazon Aurora

AWS's cloud-native, MySQL- and PostgreSQL-compatible relational database. Storage is distributed across 3 AZs with 6 copies and auto-grows to 128 TB; up to 15 low-lag read replicas, fast failover, and often quoted as up to 5x MySQL and 3x PostgreSQL performance. Variants: Serverless v2 and Global Database.

**Use case:** a PostgreSQL- or MySQL-compatible database for a busy, read-heavy application that needs fast failover.

**How to use:** create a cluster, add a writer instance, then add reader instances; applications use the cluster endpoint for writes and the reader endpoint for reads.

```bash
aws rds create-db-cluster --db-cluster-identifier app-aurora --engine aurora-postgresql \
  --master-username appadmin --manage-master-user-password

aws rds create-db-instance --db-instance-identifier app-aurora-writer \
  --db-cluster-identifier app-aurora --engine aurora-postgresql --db-instance-class db.r6g.large
```

### 21. Amazon ElastiCache

Managed in-memory caching with **Redis/Valkey** (persistence, replication, data structures, pub/sub, cluster mode) or **Memcached** (simple, multi-threaded, no persistence). Use for cache-aside, session storage, leaderboards and rate limiting to relieve databases.

**Use case:** speed up a slow database by caching hot lookups (the cache-aside pattern).

**How to use:** create a Redis/Valkey cluster, then check the cache first and fill it on a miss.

```python
import json
import redis

r = redis.Redis(host="my-cache.example.cache.amazonaws.com", port=6379, ssl=True)

def get_user(user_id):
    cached = r.get(f"user:{user_id}")
    if cached:
        return json.loads(cached)
    user = db_query_user(user_id)             # slow database call
    r.setex(f"user:{user_id}", 300, json.dumps(user))   # keep for 5 minutes
    return user
```

Use `ssl=True` only if in-transit encryption is enabled on the cluster.

## Part 3: Security, Integration and Data

### 22. AWS KMS and Secrets Manager

**KMS** manages encryption keys (customer-managed or AWS-managed), supports envelope encryption, key policies, rotation and audit via CloudTrail. **Secrets Manager** stores and automatically rotates secrets like database credentials and API keys, retrieved at runtime. A cheaper alternative for plain config is SSM Parameter Store.

**Use case:** keep a database password out of code and rotate it safely; encrypt small pieces of sensitive data.

**How to use:** store the secret, then have the application fetch it at runtime using its IAM role.

```bash
aws secretsmanager create-secret --name prod/app/db \
  --secret-string '{"username": "appadmin", "password": "change-me"}'
aws secretsmanager get-secret-value --secret-id prod/app/db --query SecretString --output text

aws kms create-key --description "app data key"
aws kms encrypt --key-id alias/app-key --plaintext fileb://data.txt \
  --query CiphertextBlob --output text
```

KMS `encrypt` handles small payloads (up to 4 KB); larger data uses envelope encryption through the SDK.

### 23. Amazon EventBridge

Serverless event bus that routes events from AWS services, SaaS apps and your own code to targets using rules and patterns. Supports schedules (cron), event archive and replay, and schema registry. The backbone of event-driven architectures.

**Use case:** run a Lambda function every night at 02:00 UTC (a scheduled report or cleanup job).

**How to use:** create a schedule rule, point it at the function, and allow EventBridge to invoke it.

```bash
aws events put-rule --name nightly-report --schedule-expression "cron(0 2 * * ? *)"
aws events put-targets --rule nightly-report \
  --targets "Id"="1","Arn"="arn:aws:lambda:us-east-1:123456789012:function:report"
aws lambda add-permission --function-name report --statement-id nightly \
  --action lambda:InvokeFunction --principal events.amazonaws.com
```

### 24. AWS Step Functions

Serverless workflow orchestration using state machines (Amazon States Language): Task, Choice, Parallel, Map, Wait, with built-in retries and error handling. **Standard** workflows run up to a year; **Express** are high-volume and short. Ideal for multi-step processes and sagas instead of chaining Lambdas.

**Use case:** an order workflow (charge, then ship) with automatic retries and clear error handling.

**How to use:** define the state machine in JSON, create it with a role, and start executions with input.

```json
{
  "StartAt": "ChargeCard",
  "States": {
    "ChargeCard": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:charge",
      "Retry": [{"ErrorEquals": ["States.TaskFailed"], "MaxAttempts": 2}],
      "Next": "ShipOrder"
    },
    "ShipOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123456789012:function:ship",
      "End": true
    }
  }
}
```

```bash
aws stepfunctions create-state-machine --name order-flow --definition file://flow.json --role-arn <role-arn>
aws stepfunctions start-execution --state-machine-arn <arn> --input '{"orderId": 42}'
```

### 25. Amazon Cognito

User authentication and authorization for apps. **User Pools** are a user directory handling sign-up/sign-in, MFA and social/SAML federation, issuing JWTs. **Identity Pools** exchange tokens for temporary AWS credentials to access services like S3 directly.

**Use case:** sign-up and sign-in for a web or mobile app without building your own user database.

**How to use:** create a user pool and an app client, then use the hosted sign-in page or an SDK. After login the app receives JWTs that your API (for example API Gateway) can validate.

```bash
aws cognito-idp create-user-pool --pool-name app-users --auto-verified-attributes email
aws cognito-idp create-user-pool-client --user-pool-id <pool-id> --client-name web --no-generate-secret
```

### 26. Amazon Kinesis

Real-time streaming data. **Data Streams**: durable shards for custom consumers. **Data Firehose**: fully managed delivery to S3, Redshift or OpenSearch with optional transformation. **Managed Service for Apache Flink**: stream analytics. Compare with SQS: Kinesis keeps ordered, replayable records; SQS deletes after processing.

**Use case:** ingest website click events in real time so several consumers (analytics, alerting) can read them.

**How to use:** create a stream, have producers put records with a partition key (records with the same key stay ordered), and read with a consumer.

```bash
aws kinesis create-stream --stream-name clicks --shard-count 1
aws kinesis put-record --stream-name clicks --partition-key user42 \
  --cli-binary-format raw-in-base64-out --data '{"page": "/home"}'
```

### 27. Amazon Redshift and Athena

**Redshift**: columnar, massively parallel data warehouse for complex analytics on structured data (provisioned or serverless; Spectrum queries S3). **Athena**: serverless SQL directly over files in S3 (Parquet, CSV, JSON), pay per data scanned, so partition and compress data to cut cost.

**Use case:** run SQL over log files sitting in S3 with Athena (no servers), and use Redshift when you need a fast warehouse for repeated dashboards.

**How to use:** in the Athena console, define a table over the S3 folder and query it. You pay per amount of data scanned, so use compressed, partitioned formats like Parquet.

```sql
CREATE EXTERNAL TABLE logs (ts string, ip string, status int)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
LOCATION 's3://my-logs-bucket/web/';

SELECT status, COUNT(*) AS hits
FROM logs
GROUP BY status
ORDER BY hits DESC;
```

### 28. AWS Elastic Beanstalk

Platform-as-a-service: upload code (Java, Node, Python, Docker, etc.) and it provisions EC2, load balancing, Auto Scaling and monitoring for you while still letting you access the resources. Good for quick deployments without deep infrastructure knowledge; less flexible than ECS/EKS.

**Use case:** deploy a small Flask or Node.js app quickly without learning the underlying AWS pieces.

**How to use:** with the EB CLI, initialize the app, create an environment, and deploy updates.

```bash
eb init -p python-3.11 my-app --region us-east-1
eb create my-app-env
eb deploy
eb open
```

### 29. AWS CodePipeline, CodeBuild, CodeDeploy

The AWS CI/CD toolchain. **CodePipeline** orchestrates stages (source, build, test, deploy). **CodeBuild** runs builds and tests in managed containers using `buildspec.yml`. **CodeDeploy** automates deployments to EC2, Lambda and ECS with in-place, rolling or blue/green strategies using `appspec` files.

**Use case:** every push to your repository is automatically built, tested and deployed.

**How to use:** create a pipeline with Source (GitHub or CodeCommit), Build (CodeBuild) and Deploy (CodeDeploy, ECS or Elastic Beanstalk) stages. CodeBuild reads its steps from `buildspec.yml` in your repo:

```yaml
version: 0.2
phases:
  install:
    runtime-versions:
      nodejs: 20
  build:
    commands:
      - npm ci
      - npm test
      - npm run build
artifacts:
  files:
    - '**/*'
  base-directory: dist
```

### 30. AWS CLI and SDKs

The **CLI** manages AWS from the terminal (`aws s3 ls`, `aws ec2 describe-instances`), configured with `aws configure` or SSO and named profiles. **SDKs** (boto3 for Python, JavaScript, Java, Go, etc.) let code call AWS APIs. On AWS compute, use IAM roles rather than hard-coded keys.

**Use case:** automate AWS from scripts, or call AWS services from application code.

**How to use:** sign in once with SSO, run CLI commands with a named profile, and use boto3 in Python.

```bash
aws configure sso
aws s3 ls --profile dev
```

```python
import boto3

s3 = boto3.client("s3")
for bucket in s3.list_buckets()["Buckets"]:
    print(bucket["Name"])
```

On EC2, Lambda or ECS the SDK picks up the attached IAM role automatically.

## Part 4: Architecture Concepts

### 31. Regions, Availability Zones and Edge Locations

A **Region** is a geographic area (e.g. `us-east-1`) with independent infrastructure. An **Availability Zone** is one or more isolated data centers within a region, connected by low-latency links; deploy across at least two for high availability. **Edge locations** are CloudFront/Route 53 points of presence. Choose regions by latency, compliance, service availability and cost.

**Use case:** place your app in the region closest to your users (and one that meets data-residency rules), spread across at least two AZs.

**How to use:** list what is available, then create subnets in different AZs.

```bash
aws ec2 describe-regions --query "Regions[].RegionName"
aws ec2 describe-availability-zones --region us-east-1 --query "AvailabilityZones[].ZoneName"
```

Example: web servers in `us-east-1a` and `us-east-1b` behind one load balancer, so one AZ failing does not take the site down.

### 32. Security Groups vs Network ACLs

**Security Groups** are instance-level firewalls: **stateful** (return traffic is automatically allowed), allow rules only, all rules evaluated. **Network ACLs** are subnet-level: **stateless** (must allow return traffic explicitly), support allow and deny, rules evaluated in numbered order. Use security groups as the main control and NACLs for coarse blocks.

**Use case:** allow HTTPS from the internet to the web tier, let only the web tier reach the database, and block a known bad address range.

**How to use:** security groups can reference other security groups; a network ACL rule blocks at the subnet.

```bash
aws ec2 authorize-security-group-ingress --group-id sg-web --protocol tcp --port 443 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-db --protocol tcp --port 5432 --source-group sg-web

aws ec2 create-network-acl-entry --network-acl-id acl-0123 --rule-number 90 \
  --protocol -1 --rule-action deny --cidr-block 203.0.113.0/24 --ingress
```

### 33. S3 storage classes and lifecycle policies

Classes trade cost for access: Standard, **Intelligent-Tiering** (automatic), Standard-IA, One Zone-IA, Glacier Instant Retrieval, Glacier Flexible Retrieval, and Glacier Deep Archive (cheapest, hours to retrieve). **Lifecycle rules** automatically transition objects between classes or expire them after N days, and clean up old versions and incomplete multipart uploads.

**Use case:** keep application logs hot for 90 days, archive them to Glacier, and delete them after a year to control cost.

**How to use:** save a lifecycle configuration and apply it to the bucket.

```json
{
  "Rules": [{
    "ID": "archive-logs",
    "Status": "Enabled",
    "Filter": {"Prefix": "logs/"},
    "Transitions": [{"Days": 90, "StorageClass": "GLACIER"}],
    "Expiration": {"Days": 365}
  }]
}
```

```bash
aws s3api put-bucket-lifecycle-configuration --bucket my-logs-bucket --lifecycle-configuration file://lifecycle.json
```

### 34. Shared Responsibility Model

AWS is responsible for security **of** the cloud (hardware, facilities, hypervisor, managed-service infrastructure). You are responsible for security **in** the cloud (data, IAM, OS patching on EC2, network configuration, encryption choices). The split shifts with the service: for example, you patch EC2 but not Lambda or RDS hosts.

**Use case:** know which security tasks are yours for each service you adopt.

**How to use:** for each service, ask who handles each layer. Typical split:

- **EC2:** AWS secures the hardware and hypervisor; you patch the OS, configure security groups, manage IAM and encrypt data.
- **RDS:** AWS patches the database host and engine versions (on your schedule); you control network access, users, parameter settings and encryption choices.
- **Lambda:** AWS manages the servers and runtime; you secure your code, dependencies and the function's IAM permissions.
- **S3:** AWS secures the service; you decide bucket policies, public access settings and encryption.

### 35. AWS Well-Architected Framework

Best-practice guidance across six pillars: **Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability**. Used for architecture reviews through the Well-Architected Tool. In interviews, justify design choices by naming the pillar and trade-off.

**Use case:** review a workload before launch, or after an incident, to find the biggest risks.

**How to use:**

1. Open the AWS Well-Architected Tool in the console and define your workload.
2. Answer the questions for each pillar (for example, "How do you manage identities?" under Security).
3. Review the high- and medium-risk items it flags and the improvement plan.
4. Fix the top items, then re-run the review later to track progress.

Example finding: a single-AZ database is flagged under Reliability, and the fix is enabling Multi-AZ.

### 36. Cost management (Cost Explorer, Budgets, pricing models)

**Cost Explorer** visualizes and forecasts spend; **Budgets** alerts you when cost or usage crosses thresholds; Cost and Usage Reports give detail; tags enable cost allocation. Savings levers: right-sizing, Savings Plans and Reserved Instances, Spot, S3 lifecycle and storage classes, turning off idle resources, and cutting data transfer. Data transfer **out** and NAT Gateway traffic are common surprise costs.

**Use case:** find out which service is driving your bill and get warned before spend passes a limit.

**How to use:** group last month's cost by service with Cost Explorer, then create a budget with an email alert in the console (Billing, then Budgets).

```bash
aws ce get-cost-and-usage --time-period Start=2026-10-01,End=2026-11-01 \
  --granularity MONTHLY --metrics UnblendedCost \
  --group-by Type=DIMENSION,Key=SERVICE
```

Tag resources (for example `team=checkout`) and enable those tags for cost allocation so you can break costs down by owner.

## Part 5: Additional Services

Services that interviews and real projects often reach for, grouped as containers and storage, security and governance, operations, data and AI, and hybrid networking.

### 37. Amazon ECR

A managed, private **container image registry** (like a private Docker Hub). ECS, EKS, Lambda and Fargate pull images from it using IAM permissions. Features: image scanning for vulnerabilities, lifecycle policies that delete old images, immutable tags, cross-region replication and encryption at rest.

**Use case:** store your application's Docker images so ECS or EKS can deploy them.

**How to use:** create a repository, log Docker in with a short-lived token, then tag and push.

```bash
aws ecr create-repository --repository-name my-app
aws ecr get-login-password --region us-east-1 | docker login --username AWS \
  --password-stdin 123456789012.dkr.ecr.us-east-1.amazonaws.com
docker tag my-app:latest 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
docker push 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
```

### 38. Amazon EFS and FSx

**EFS** is a managed, elastic **NFS file system** that many Linux instances, containers or Lambda functions can mount at the same time across AZs; it grows and shrinks automatically. Contrast with EBS (one instance, one AZ) and S3 (objects, not a filesystem). **FSx** offers managed specialty file systems: FSx for Windows File Server (SMB), FSx for Lustre (high-performance computing and ML training), FSx for NetApp ONTAP and OpenZFS.

**Use case:** shared uploads or content that several web servers must all see.

**How to use:** create the file system, add a mount target in each subnet, then mount it on instances (needs the `amazon-efs-utils` package).

```bash
aws efs create-file-system --throughput-mode elastic --tags Key=Name,Value=shared-files
aws efs create-mount-target --file-system-id fs-0123 --subnet-id subnet-aaa --security-groups sg-0123

sudo mkdir /mnt/shared
sudo mount -t efs fs-0123:/ /mnt/shared
```

### 39. AWS CloudTrail

Records **API calls and console actions** in your account (who did what, from where, and when). It answers audit and security questions such as "who deleted this bucket?". Management events are logged by default for 90 days in Event history; create a **trail** to keep them long-term in S3 and optionally send them to CloudWatch Logs. Enable multi-region trails and log file validation. Do not confuse it with CloudWatch (performance metrics and logs).

**Use case:** investigate an unexpected change, and satisfy compliance audits.

**How to use:** create a trail that writes to an S3 bucket (the bucket needs a policy allowing CloudTrail), start logging, then look up events.

```bash
aws cloudtrail create-trail --name org-trail --s3-bucket-name my-trail-logs --is-multi-region-trail
aws cloudtrail start-logging --name org-trail
aws cloudtrail lookup-events --lookup-attributes AttributeKey=EventName,AttributeValue=DeleteBucket --max-results 5
```

### 40. AWS Config

Continuously **records the configuration of your resources** and evaluates it against **rules**, showing compliance and a history of how each resource changed over time. Use managed rules (for example, "S3 buckets must not allow public read") or custom rules backed by Lambda, and auto-remediate with Systems Manager. Difference from CloudTrail: CloudTrail records *actions*, Config records *state and drift*.

**Use case:** prove that all buckets are encrypted and flag any that are public.

**How to use:** turn on the Config recorder, add a managed rule, and check compliance.

```json
{"ConfigRuleName": "s3-no-public-read",
 "Source": {"Owner": "AWS", "SourceIdentifier": "S3_BUCKET_PUBLIC_READ_PROHIBITED"}}
```

```bash
aws configservice put-config-rule --config-rule file://rule.json
aws configservice describe-compliance-by-config-rule --config-rule-names s3-no-public-read
```

### 41. AWS Organizations and SCPs

**Organizations** lets you manage many AWS accounts centrally: group them into **organizational units (OUs)**, use consolidated billing, and share resources. **Service Control Policies (SCPs)** set the maximum permissions accounts in an OU or the organization can ever use. An SCP never grants access; it only limits what IAM can allow, and it does not apply to the management account. Multi-account setups (separate dev, test, prod and security accounts) limit blast radius, and **AWS Control Tower** automates a well-governed starting structure.

**Use case:** make sure nobody in any account can switch off CloudTrail.

**How to use:** write an SCP, create it, and attach it to an OU.

```json
{"Version": "2012-10-17",
 "Statement": [{"Effect": "Deny",
   "Action": ["cloudtrail:StopLogging", "cloudtrail:DeleteTrail"],
   "Resource": "*"}]}
```

```bash
aws organizations create-policy --name protect-cloudtrail --type SERVICE_CONTROL_POLICY \
  --description "Prevent turning off CloudTrail" --content file://scp.json
aws organizations attach-policy --policy-id p-examplepolicy --target-id ou-examp-12345678
```

### 42. AWS WAF and AWS Shield

**AWS WAF** is a web application firewall that filters HTTP requests reaching CloudFront, an Application Load Balancer, API Gateway or AppSync. Rules block or rate-limit patterns such as SQL injection, cross-site scripting, bad IPs or too many requests, and **managed rule groups** give you ready-made protections. **AWS Shield** protects against DDoS: **Standard** is automatic and free for all customers; **Advanced** is a paid tier with extra detection, response-team support and cost protection.

**Use case:** protect a public web app from common attacks and abusive traffic.

**How to use:** create a web ACL with an AWS managed rule group, then attach it to the load balancer.

```bash
aws wafv2 create-web-acl --name web-acl --scope REGIONAL --default-action Allow={} \
  --visibility-config SampledRequestsEnabled=true,CloudWatchMetricsEnabled=true,MetricName=webacl \
  --rules file://rules.json
aws wafv2 associate-web-acl --web-acl-arn <web-acl-arn> --resource-arn <alb-arn>
```

`rules.json`:

```json
[{"Name": "AWSCommon", "Priority": 0,
  "Statement": {"ManagedRuleGroupStatement": {"VendorName": "AWS", "Name": "AWSManagedRulesCommonRuleSet"}},
  "OverrideAction": {"None": {}},
  "VisibilityConfig": {"SampledRequestsEnabled": true, "CloudWatchMetricsEnabled": true, "MetricName": "AWSCommon"}}]
```

### 43. Amazon GuardDuty

Managed **threat detection** that analyzes CloudTrail events, VPC flow logs and DNS logs (plus S3, EKS and other sources when enabled) using threat intelligence and machine learning to flag suspicious behavior: compromised credentials, crypto-mining instances, communication with known malicious IPs, or unusual API calls. It needs no agents and findings can trigger EventBridge rules for automated response. For broader aggregation, **Security Hub** collects findings from several services.

**Use case:** get alerted when an instance starts talking to a known malicious address.

**How to use:** enable a detector in each region, then review findings or route them to EventBridge or SNS.

```bash
aws guardduty create-detector --enable
aws guardduty list-findings --detector-id <detector-id>
aws guardduty get-findings --detector-id <detector-id> --finding-ids <finding-id>
```

### 44. AWS Systems Manager

A toolbox for operating fleets of servers (EC2 and on-premises) through the SSM agent. Key capabilities: **Session Manager** (shell access with no open SSH port or key pairs, with audit logs), **Run Command** (run a command on many instances), **Patch Manager**, **Parameter Store** (hierarchical configuration and secrets, a cheaper alternative to Secrets Manager for plain config) and **Automation** runbooks. Instances need the SSM agent and an IAM role that allows Systems Manager.

**Use case:** log into a private server without opening port 22, and keep configuration out of code.

**How to use:**

```bash
aws ssm start-session --target i-0123456789abcdef0

aws ssm put-parameter --name /prod/app/api-url --value https://api.example.com --type String
aws ssm get-parameter --name /prod/app/api-url --query Parameter.Value --output text

aws ssm send-command --document-name AWS-RunShellScript \
  --targets Key=tag:Env,Values=prod --parameters 'commands=["uptime"]'
```

### 45. AWS Glue

A serverless **data integration (ETL) service**. A **crawler** scans data (for example files in S3) and records its schema in the **Glue Data Catalog**, which Athena, Redshift Spectrum and EMR can all use. **Glue jobs** (Spark or Python) clean and transform data, and **workflows/triggers** schedule them. It is the usual glue between a raw S3 data lake and analytics tools.

**Use case:** turn raw CSV logs in S3 into a queryable table.

**How to use:** create and run a crawler, then query the discovered table in Athena.

```bash
aws glue create-crawler --name logs-crawler --role <glue-role-arn> --database-name weblogs \
  --targets '{"S3Targets": [{"Path": "s3://my-logs-bucket/web/"}]}'
aws glue start-crawler --name logs-crawler
aws glue get-tables --database-name weblogs
```

### 46. Amazon OpenSearch Service

Managed **OpenSearch** (the open-source fork of Elasticsearch): a distributed search and analytics engine built on inverted indexes. Common uses are full-text search for applications, **log and observability analytics** (with OpenSearch Dashboards), and vector search for semantic or RAG workloads. Deploy as managed domains or as a serverless collection. It is a search index, not a system of record: keep your source of truth in a database such as DynamoDB or RDS.

**Use case:** add fast text search to a product catalog or search application logs.

**How to use:** create a domain, put it behind fine-grained access control and a VPC, then index and query documents.

```bash
aws opensearch create-domain --domain-name logs --engine-version OpenSearch_2.11 \
  --cluster-config InstanceType=t3.small.search,InstanceCount=1 \
  --ebs-options EBSEnabled=true,VolumeType=gp3,VolumeSize=20
```

```json
PUT /products/_doc/1
{"name": "Trail running shoes", "price": 89}

GET /products/_search
{"query": {"match": {"name": "running"}}}
```

### 47. Amazon Bedrock

A fully managed service for building **generative AI applications** through one API to foundation models from multiple providers (including Anthropic's Claude). Features: a unified Converse API, **Knowledge Bases** (managed RAG over your documents), **Agents** (tool-using workflows), **Guardrails** (content and PII filters), fine-tuning on supported models, and private access over a VPC endpoint. Your prompts and data are not used to train the base models. You must request access to each model in your account and region first.

**Use case:** add summarization or a chatbot to an app without hosting a model.

**How to use:** call the model with the SDK (the model ID is a placeholder for one you have enabled).

```python
import boto3

client = boto3.client("bedrock-runtime", region_name="us-east-1")
response = client.converse(
    modelId="<model-id>",
    messages=[{"role": "user", "content": [{"text": "Explain RAG in two sentences."}]}],
    inferenceConfig={"maxTokens": 200, "temperature": 0.3},
)
print(response["output"]["message"]["content"][0]["text"])
```

### 48. Amazon SageMaker

AWS's platform for the **full machine-learning lifecycle**: **Studio** notebooks, managed **training jobs** (including distributed and spot training), **real-time, serverless and batch inference endpoints**, **JumpStart** (pretrained models), **Pipelines** (ML CI/CD), a **Model Registry**, **Feature Store**, **Clarify** (bias and explainability) and **Model Monitor** (drift). Choose Bedrock when you want to consume a hosted foundation model through an API; choose SageMaker when you train, customize or host your own models.

**Use case:** train a custom model on data in S3 and serve it behind an endpoint.

**How to use:** with the SageMaker Python SDK, launch a training job and deploy it. Delete the endpoint when done because it bills while running.

```python
from sagemaker.estimator import Estimator

estimator = Estimator(
    image_uri="<training-image-uri>",
    role="<sagemaker-role-arn>",
    instance_count=1,
    instance_type="ml.m5.xlarge",
    output_path="s3://my-ml-bucket/output",
)
estimator.fit({"train": "s3://my-ml-bucket/train/"})

predictor = estimator.deploy(initial_instance_count=1, instance_type="ml.m5.large")
result = predictor.predict(payload)
predictor.delete_endpoint()
```

### 49. Hybrid Networking: Site-to-Site VPN, Direct Connect and Transit Gateway

Connect AWS to your office or data center and to other VPCs. **Site-to-Site VPN** is an encrypted IPsec tunnel over the internet: quick to set up and cheap, but variable latency. **Direct Connect** is a dedicated private fiber link: consistent low latency and higher bandwidth, but takes weeks to provision and is not encrypted by default (add a VPN or MACsec on top). **Transit Gateway** is a central hub that connects many VPCs and on-premises networks, replacing a messy mesh of VPC peering connections. **VPC peering** is simple one-to-one connectivity and is not transitive.

**Use case:** let an on-premises system reach private databases in your VPC, and connect several VPCs through one hub.

**How to use:**

```bash
aws ec2 create-customer-gateway --type ipsec.1 --public-ip 203.0.113.10 --bgp-asn 65000
aws ec2 create-vpn-connection --type ipsec.1 --customer-gateway-id cgw-0123 --transit-gateway-id tgw-0123

aws ec2 create-transit-gateway --description "central hub"
aws ec2 create-transit-gateway-vpc-attachment --transit-gateway-id tgw-0123 \
  --vpc-id vpc-0123 --subnet-ids subnet-aaa subnet-bbb
```

---

## Practice Prompts

1. Design a highly available three-tier web app on AWS (VPC, subnets, ALB, Auto Scaling, RDS Multi-AZ).
2. When would you pick Lambda over ECS Fargate, and vice versa?
3. A Lambda reads from SQS but messages are processed twice. Why and how to handle it? (Answer: at-least-once delivery, so make processing idempotent.)
4. How do you give an EC2 instance access to S3 securely? (IAM role, not access keys.)
5. How would you cut a surprising AWS bill?
