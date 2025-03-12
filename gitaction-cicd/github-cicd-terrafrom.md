🚀 Automating Terraform Deployment to AWS ECS with GitHub Actions
Now, we'll integrate GitHub Actions with Terraform to automatically deploy ECS infrastructure and push updates to AWS whenever we push code.

📌 Overview
With this GitHub Actions workflow, we will:

✅ Run Terraform to deploy/update AWS ECS.

✅ Build and Push Docker Image to AWS ECR.

✅ Trigger ECS Service Update to use the latest Docker image.

🛠 Step 1: Store AWS Credentials in GitHub Secrets
Go to GitHub Repository → Settings → Actions → Secrets and add the following secrets:

Secret Name	Value
AWS_ACCESS_KEY_ID	Your AWS Access Key
AWS_SECRET_ACCESS_KEY	Your AWS Secret Key
AWS_REGION	Example: us-east-1
ECR_REPOSITORY	Example: flask-ecs-app
ECS_CLUSTER	Example: flask-cluster
ECS_SERVICE	Example: flask-service
📄 Step 2: Create GitHub Actions Workflow
Inside your project root, create a new directory:
sh
Copy code
mkdir -p .github/workflows
Create a Terraform GitHub Actions Workflow:
sh
Copy code
nano .github/workflows/terraform-deploy.yml
Paste the following GitHub Actions YAML:
📌 GitHub Actions Terraform Pipeline (terraform-deploy.yml)
yaml
Copy code
name: Deploy Terraform and ECS

# Run workflow on every push to main branch
on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Deploy Terraform and ECS
    runs-on: ubuntu-latest

    steps:
    # 🔹 Step 1: Checkout Code
    - name: Checkout Repository
      uses: actions/checkout@v3

    # 🔹 Step 2: Configure AWS Credentials
    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ secrets.AWS_REGION }}
    
    # 🔹 Step 3: Install Terraform
    - name: Install Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: latest

    # 🔹 Step 4: Initialize Terraform
    - name: Terraform Init
      run: terraform init

    # 🔹 Step 5: Plan Terraform Changes
    - name: Terraform Plan
      run: terraform plan

    # 🔹 Step 6: Apply Terraform Changes
    - name: Terraform Apply
      run: terraform apply -auto-approve

    # 🔹 Step 7: Authenticate Docker with AWS ECR
    - name: Login to Amazon ECR
      run: |
        aws ecr get-login-password --region ${{ secrets.AWS_REGION }} | \
        docker login --username AWS --password-stdin \
        $(aws ecr describe-repositories --query "repositories[0].repositoryUri" --output text)

    # 🔹 Step 8: Build and Tag Docker Image
    - name: Build and Tag Docker Image
      run: |
        IMAGE_TAG=$(date +%s)
        docker build -t flask-ecs-app:$IMAGE_TAG .
        docker tag flask-ecs-app:$IMAGE_TAG \
        $(aws ecr describe-repositories --query "repositories[0].repositoryUri" --output text):$IMAGE_TAG
        echo "IMAGE_TAG=$IMAGE_TAG" >> $GITHUB_ENV

    # 🔹 Step 9: Push Docker Image to ECR
    - name: Push Docker Image to ECR
      run: |
        docker push $(aws ecr describe-repositories --query "repositories[0].repositoryUri" --output text):$IMAGE_TAG

    # 🔹 Step 10: Force ECS Service to Use New Image
    - name: Deploy New Image to ECS
      run: |
        aws ecs update-service \
        --cluster ${{ secrets.ECS_CLUSTER }} \
        --service ${{ secrets.ECS_SERVICE }} \
        --force-new-deployment
🚀 Step 3: Commit and Push Terraform & Workflow File
Now, commit and push the GitHub Actions Terraform configuration:

sh
Copy code
git add .
git commit -m "Added GitHub Actions for Terraform & ECS deployment"
git push origin main
✅ Step 4: Monitor GitHub Actions
Go to GitHub Repository → Actions Tab.
You should see the Terraform Deploy Job running.
If successful:
An ECS Cluster will be created & updated.
The Docker Image will be built, pushed to ECR.
The ECS Service will be updated to use the new image.
🎯 Step 5: Verify Everything in AWS
1️⃣ Check Infrastructure with Terraform:

sh
Copy code
terraform state list
This will list all AWS resources Terraform is managing.

2️⃣ Check AWS ECS Console:

Navigate to AWS Console → ECS → flask-cluster → Services
Click on flask-service
Verify that a new task is running with the latest image.
3️⃣ Get Public IP & Test:

sh
Copy code
aws ecs describe-tasks --cluster flask-cluster --tasks <TASK_ID>
Find the public IP and open:

javascript
Run Code
Copy code
http://<PUBLIC_IP>:5000
You should see:

json
Copy code
{"message": "Hello from ECS!"}
🎯 Next Steps
✅ Integrate Code Testing Before Deployment

✅ Implement Blue-Green Deployment Strategy

✅ Attach an AWS Load Balancer for High Availability

✅ Set Up Monitoring with AWS CloudWatch

Would you like to automate Terraform destruction for cleanup or explore load balancer configurations next? Let me know! 🚀😊
