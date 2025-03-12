Certainly! Below is a complete CI/CD pipeline setup using GitHub Actions to automate deploying an application to Google Cloud Run on GCP.

Prerequisites
Before setting up the pipeline:

Enable Cloud Run:

Ensure that Cloud Run is enabled on your GCP Project.
Create a Service Account:

Go to Google Cloud Console → IAM & Admin → Service Accounts.
Create a new service account with Cloud Run Admin, Cloud Build Editor, Storage Admin, and Viewer roles.
Generate a JSON key for this service account.
Store GitHub Secrets:

Go to GitHub Repository → Settings → Secrets and Variables → Actions.
Add the following secrets:
GCP_PROJECT_ID → Your Google Cloud Project ID.
GCP_SA_KEY → The service account JSON key (Base64 encoded).
CI/CD Pipeline using GitHub Actions
Save the following workflow file as .github/workflows/deploy.yaml.

deploy.yaml
yaml
Copy code
name: Deploy to Cloud Run

on:
  push:
    branches:
      - main  # Runs only when pushing to main branch

jobs:
  deploy:
    name: Deploy to GCP Cloud Run
    runs-on: ubuntu-latest

    steps:
    # Step 1: Checkout the code
    - name: Checkout repository
      uses: actions/checkout@v4

    # Step 2: Authenticate with GCP
    - name: Authenticate with Google Cloud
      uses: google-github-actions/auth@v2
      with:
        credentials_json: ${{ secrets.GCP_SA_KEY }}

    # Step 3: Setup gcloud CLI
    - name: Set up Google Cloud SDK
      uses: google-github-actions/setup-gcloud@v2
      with:
        project_id: ${{ secrets.GCP_PROJECT_ID }}

    # Step 4: Build and push Docker image to Google Container Registry (GCR)
    - name: Build and Push Docker Image
      run: |
        gcloud auth configure-docker gcr.io
        IMAGE_NAME=gcr.io/${{ secrets.GCP_PROJECT_ID }}/my-app:${{ github.sha }}
        docker build -t $IMAGE_NAME .
        docker push $IMAGE_NAME

    # Step 5: Deploy to Cloud Run
    - name: Deploy to Cloud Run
      run: |
        IMAGE_NAME=gcr.io/${{ secrets.GCP_PROJECT_ID }}/my-app:${{ github.sha }}
        gcloud run deploy my-app \
          --image $IMAGE_NAME \
          --platform managed \
          --region us-central1 \
          --allow-unauthenticated

    # Step 6: Verify Deployment
    - name: Get Cloud Run Service URL
      run: |
        gcloud run services describe my-app --region us-central1 --format 'value(status.url)'
Explanation of the Workflow
Triggers on Push to main.
Authenticate to GCP using the service account key.
Setup gcloud CLI to interact with Google Cloud.
Build & Push Docker Image to Google Container Registry (GCR).
Deploy the Image to Cloud Run.
Retrieve the Live URL of the running service.
Final Steps
After committing this file to your repository:

Push to the main branch.
The GitHub Actions workflow will automatically build, push, and deploy your app 🚀.
Visit https://my-app-xxxxxx-uc.a.run.app (retrieved from gcloud run services describe).
✅ Your app is now continuously deployed on Google Cloud Run! 🎉 🚀
