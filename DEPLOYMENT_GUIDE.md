# AWS ECS Deployment Guide

This guide walks you through deploying the Maintenance Page to AWS ECS (Elastic Container Service).

## Prerequisites

- AWS Account with appropriate permissions
- AWS CLI installed and configured
- Docker installed locally
- ECR (Elastic Container Registry) access
- ECS cluster created
- Application Load Balancer (optional, for production)

## Step 1: Prepare Docker Image

### 1.1 Build Docker Image Locally

```bash
# Navigate to project directory
cd "NEW Maintance Page"

# Build the Docker image
docker build -t maintenance-page:latest .
```

### 1.2 Test Locally

```bash
# Run container locally
docker run -d -p 8080:80 --name maintenance-test maintenance-page:latest

# Test in browser
# Open http://localhost:8080

# Stop and remove test container
docker stop maintenance-test
docker rm maintenance-test
```

## Step 2: Push to Amazon ECR

### 2.1 Create ECR Repository

```bash
# Create repository
aws ecr create-repository --repository-name maintenance-page --region us-east-1

# Note the repository URI (e.g., 123456789012.dkr.ecr.us-east-1.amazonaws.com/maintenance-page)
```

### 2.2 Authenticate Docker to ECR

```bash
# Get login token
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com

# Replace <account-id> with your AWS account ID
```

### 2.3 Tag and Push Image

```bash
# Get your ECR repository URI
ECR_URI=<account-id>.dkr.ecr.us-east-1.amazonaws.com/maintenance-page

# Tag the image
docker tag maintenance-page:latest $ECR_URI:latest

# Push to ECR
docker push $ECR_URI:latest
```

## Step 3: Create ECS Task Definition

### 3.1 Create task-definition.json

Create a file `task-definition.json`:

```json
{
  "family": "maintenance-page",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "containerDefinitions": [
    {
      "name": "maintenance-page",
      "image": "<account-id>.dkr.ecr.us-east-1.amazonaws.com/maintenance-page:latest",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/maintenance-page",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

### 3.2 Register Task Definition

```bash
aws ecs register-task-definition --cli-input-json file://task-definition.json --region us-east-1
```

## Step 4: Create ECS Service

### 4.1 Create CloudWatch Log Group

```bash
aws logs create-log-group --log-group-name /ecs/maintenance-page --region us-east-1
```

### 4.2 Create ECS Service

```bash
aws ecs create-service \
  --cluster your-cluster-name \
  --service-name maintenance-page-service \
  --task-definition maintenance-page \
  --desired-count 1 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-xxx,subnet-yyy],securityGroups=[sg-xxx],assignPublicIp=ENABLED}" \
  --region us-east-1
```

**Note**: Replace:
- `your-cluster-name` with your ECS cluster name
- `subnet-xxx, subnet-yyy` with your subnet IDs
- `sg-xxx` with your security group ID (must allow inbound port 80)

## Step 5: Configure Application Load Balancer (Optional but Recommended)

### 5.1 Create Target Group

```bash
aws elbv2 create-target-group \
  --name maintenance-page-tg \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-xxx \
  --target-type ip \
  --health-check-path / \
  --region us-east-1
```

### 5.2 Update Service with Load Balancer

```bash
aws ecs update-service \
  --cluster your-cluster-name \
  --service maintenance-page-service \
  --load-balancers "targetGroupArn=arn:aws:elasticloadbalancing:us-east-1:xxx:targetgroup/maintenance-page-tg/xxx,containerName=maintenance-page,containerPort=80" \
  --region us-east-1
```

### 5.3 Create Listener Rule

```bash
aws elbv2 create-listener \
  --load-balancer-arn <alb-arn> \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=<target-group-arn> \
  --region us-east-1
```

## Step 6: Update Image (Future Deployments)

When you need to update the maintenance page:

```bash
# 1. Build new image
docker build -t maintenance-page:latest .

# 2. Tag and push
ECR_URI=<account-id>.dkr.ecr.us-east-1.amazonaws.com/maintenance-page
docker tag maintenance-page:latest $ECR_URI:latest
docker push $ECR_URI:latest

# 3. Force new deployment
aws ecs update-service \
  --cluster your-cluster-name \
  --service maintenance-page-service \
  --force-new-deployment \
  --region us-east-1
```

## Step 7: Access Your Application

### With Load Balancer:
- Use the Load Balancer DNS name from AWS Console
- Example: `http://maintenance-page-alb-xxx.us-east-1.elb.amazonaws.com`

### Without Load Balancer:
- Get the public IP from the ECS task:
```bash
aws ecs describe-tasks \
  --cluster your-cluster-name \
  --tasks <task-id> \
  --region us-east-1
```
- Access via: `http://<public-ip>`

## Troubleshooting

### Check Task Status
```bash
aws ecs describe-services \
  --cluster your-cluster-name \
  --services maintenance-page-service \
  --region us-east-1
```

### View Logs
```bash
aws logs tail /ecs/maintenance-page --follow --region us-east-1
```

### Common Issues

1. **Task fails to start**: Check security group allows inbound port 80
2. **Image pull errors**: Verify ECR authentication and image URI
3. **502 Bad Gateway**: Check target group health checks and container port
4. **Container exits**: Check CloudWatch logs for errors

## Cost Optimization

- Use Fargate Spot for non-production (up to 70% savings)
- Set up auto-scaling based on CPU/memory
- Use CloudFront CDN for global distribution
- Enable ALB idle timeout for cost savings

## Security Best Practices

1. Use HTTPS with SSL certificate (ACM)
2. Restrict security group to specific IPs if needed
3. Enable CloudWatch logging
4. Use IAM roles with least privilege
5. Regularly update base images

## Monitoring

- CloudWatch Metrics: CPU, Memory, Request count
- CloudWatch Logs: Container logs
- ALB Access Logs: HTTP request logs
- ECS Service Events: Deployment status

## Rollback Procedure

If deployment fails:

```bash
# Update service to previous task definition
aws ecs update-service \
  --cluster your-cluster-name \
  --service maintenance-page-service \
  --task-definition maintenance-page:<previous-revision> \
  --region us-east-1
```

## Support

For issues or questions:
- Check AWS ECS documentation
- Review CloudWatch logs
- Verify task definition and service configuration

