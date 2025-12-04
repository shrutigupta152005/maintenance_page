# Quick Start Guide

## Local Testing

```bash
# Option 1: Direct browser
# Just open index.html in your browser

# Option 2: Local server
python -m http.server 8000
# Then open http://localhost:8000
```

## Docker Quick Test

```bash
docker build -t maintenance-page .
docker run -d -p 8080:80 --name test maintenance-page
# Open http://localhost:8080
```

## AWS ECS Deployment (Quick Steps)

1. **Build & Push to ECR:**
```bash
docker build -t maintenance-page .
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com
docker tag maintenance-page:latest <account-id>.dkr.ecr.us-east-1.amazonaws.com/maintenance-page:latest
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/maintenance-page:latest
```

2. **Create Task Definition:**
```bash
aws ecs register-task-definition --cli-input-json file://task-definition.json
```

3. **Create Service:**
```bash
aws ecs create-service --cluster your-cluster --service-name maintenance-page-service --task-definition maintenance-page --launch-type FARGATE --network-configuration "awsvpcConfiguration={subnets=[subnet-xxx],securityGroups=[sg-xxx],assignPublicIp=ENABLED}"
```

For detailed instructions, see `DEPLOYMENT_GUIDE.md`

