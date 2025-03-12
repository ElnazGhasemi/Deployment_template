🛠 Step 1: Create an IAM User for GitHub Actions
We'll create an IAM user in AWS to allow GitHub Actions to interact with ECS.

Go to AWS IAM Console → Users → Create User.
Name it github-actions-user.
Attach these IAM Policies:
AmazonEC2ContainerRegistryFullAccess
AmazonECS_FullAccess
Click "Create User" and copy the Access Key ID and Secret Access Key.
🔹 Now, store these credentials in GitHub Secrets.

🤖 Step 2: Set Up GitHub Secrets
Go to your GitHub Repository → Settings → Secrets → Actions, and add:

Secret Name	Value
AWS_ACCESS_KEY_ID	Your IAM User Access Key ID
AWS_SECRET_ACCESS_KEY	Your IAM User Secret Access Key
AWS_REGION	Example: us-east-1
ECR_REPOSITORY	Example: flask-ecs-app
ECS_CLUSTER	Example: flask-cluster
ECS_SERVICE	Example: flask-service
ECS_TASK_DEFINITION	Example: flask-task.json
⚡ Step 3: Create GitHub Actions Workflow
Inside your project, create the GitHub Actions CI/CD pipeline.

Navigate to your project root.
Create a .github/workflows directory:
sh
Copy code
mkdir -p .github/workflows
Create a workflow file called deploy.yml:
sh
Copy code
nano .github/workflows/deploy.yml
Paste this YAML configuration:
yaml
Copy code
name: Deploy to AWS ECS

on:
  push:
    branches:
      - main  # Deploy only when code is pushed to the main branch

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Login to Amazon ECR
        run: |
          aws ecr get-login-password --region ${{ secrets.AWS_REGION }} | \
          docker login --username AWS --password-stdin \
          ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com

      - name: Build and Tag Docker Image
        run: |
          IMAGE_TAG=$(date +%s)
          docker build -t ${{ secrets.ECR_REPOSITORY }}:$IMAGE_TAG .
          docker tag ${{ secrets.ECR_REPOSITORY }}:$IMAGE_TAG \
          ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com/${{ secrets.ECR_REPOSITORY }}:$IMAGE_TAG
          echo "IMAGE_TAG=$IMAGE_TAG" >> $GITHUB_ENV

      - name: Push Docker Image to ECR
        run: |
          docker push ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com/${{ secrets.ECR_REPOSITORY }}:$IMAGE_TAG

      - name: Update ECS Task Definition
        run: |
          aws ecs update-service \
            --cluster ${{ secrets.ECS_CLUSTER }} \
            --service ${{ secrets.ECS_SERVICE }} \
            --force-new-deployment
🚀 Step 4: Push Code and Trigger Deployment
Now, commit and push your changes to GitHub:

sh
Copy code
git add .
git commit -m "Setup CI/CD pipeline with GitHub Actions"
git push origin main
✅ Step 5: Verify Deployment
Go to your GitHub Repository → "Actions" Tab.

You should see your workflow running under "Deploy to AWS ECS."

Once it completes:

Go to AWS ECS Console.
Navigate to your ECS Cluster → flask-service.
Verify that a new task is running with the latest image.
🎉 Your Python app is now automatically deploying every time you push code! 🚀

🛠 Next Steps
✅ Automate Blue-Green deployments

✅ Use Terraform or AWS CDK for infrastructure as code

✅ Add monitoring with AWS CloudWatch and Alerts
