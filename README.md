# Secure CI/CD Pipeline for Broken Crystals Application
## This repository implements a secure and automated CI/CD pipeline using GitHub Actions based on the provided YAML configuration. The pipeline automates building, testing, and scanning your code with SonarQube for static application security testing (SAST), and prepares for deployment steps.

### Table of Contents
1. Overview
2. Prerequisites
3. Repository Setup
4. Pipeline Configuration
5. Pipeline Jobs and Steps
6. Test Job
7. Deploy Job
8. Secrets and Environment Variables
9. Running the Pipeline
10. Troubleshooting
11. References
    
### Overview
This repository contains a GitHub Actions workflow (.github/workflows/ci-cd-pipeline.yml) that defines a CI/CD pipeline for your application. The pipeline includes:

. Code Checkout: Cloning the repository to the build environment.

. Static Analysis: Using SonarQube to perform static analysis of the code to identify potential issues and vulnerabilities.

. Deploy Job (Commented Out): Steps to deploy your application using Docker containers and environment variables, including cleaning up old containers and images.

## Prerequisites
Before running this pipeline, ensure you have:

. A GitHub repository hosting your application code.

. A SonarQube instance (cloud or on-premises) for static code analysis.

. Docker installed and configured locally if you intend to run deployment steps locally.

. Secrets (like SONAR_TOKEN and SONAR_HOST_URL) stored in your repository's settings.

## Repository Setup
1. Clone the Repository (if not already done):

`git clone https://github.com/your-username/your-repository.git`

`cd your-repository`

2. Create or Update the `.github/workflows/ci-cd-pipeline.yml` File:
   
. Ensure that the file exists and contains the correct pipeline configuration as provided.

3. Set Up Required Secrets:

. Add secrets to your GitHub repository (go to Settings > Secrets and variables > Actions):

. `SONAR_TOKEN`: Your SonarQube authentication token.

. `SONAR_HOST_URL`: The URL to your SonarQube server (e.g., https://sonarcloud.io or http://localhost:9000).

## Pipeline Configuration
This repository uses the GitHub Actions workflow file located at `.github/workflows/ci-cd-pipeline.yml.` The key configuration details are:

. **Workflow Name**: `Test`

. Jobs:
1. `test`: Builds and tests your code, and runs SonarQube analysis.
   
2. `deploy`: Prepares your application for deployment (currently commented out).
An example pipeline file is shown below (updated and annotated):

`name`: Test

 This section can be uncommented to trigger the workflow on push to the `main` branch
 ```yml
 on:
   push:
     branches:
      - main

jobs:
  test:
    name: Test Job
    runs-on: self-hosted
    permissions: read-all
    steps:
      - name: Checkout Code
        uses: actions/checkout@v2
        with:
          fetch-depth: 0  # Ensures full history is fetched for better analysis relevance

      - name: Set up SonarQube Scanner
        uses: sonarsource/sonarqube-scan-action@master
        with:
          projectBaseDir: .
          # The sonar.projectKey should be unique and match your project key in SonarQube
          args: >
            -Dsonar.projectKey=broken_crystal
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
  
   The deploy job is currently commented out. Uncomment and configure the job once ready to deploy.
   deploy:
     runs-on: self-hosted # Using a self-hosted runner
     needs: test
     steps:
       - name: Clean Up Directory
         run: sudo rm -rf /home/ubuntu/actions-runner/_work/backend//*
    
       - name: Checkout Code
         uses: actions/checkout@v2

       - name: Check Current Directory
         run: pwd

       - name: Check Running Docker Services
         run: sudo docker ps

       - name: Stop Docker Containers
         run: sudo docker stop $(sudo docker ps -a -q) || true
    
       - name: Remove Docker Containers
         run: sudo docker rm $(sudo docker ps -a -q) || true

       - name: Remove Docker Images
         run: sudo docker rmi $(sudo docker images -q) || true

       - name: Retrieve Secrets
         run: echo ${{ secrets.GOOGLE_MAPS_API }} > .env

       - name: Run Docker Containers
         run: sudo docker-compose --file=docker-compose.local.yml up -d --build
```
  
## Pipeline Jobs and Steps
### Test Job
The test job is responsible for:

1. Checking Out the Code: Uses actions/checkout@v2 to retrieve your repository's code.

2. SonarQube Analysis:
. Uses the `sonarsource/sonarqube-scan-action` to analyze your code with SonarQube.

. Requires `SONAR_TOKEN` and `SONAR_HOST_URL` to be defined as secrets in your repository.

The result is a report of code smells, bugs, and vulnerabilities found in your code.

### Deploy Job
Note: The deploy job is currently commented out. When enabled, it will:

1. Prepare and Clean Up the Environment:
   . Removes old Docker containers and images to ensure a clean environment.
2. Checkout the Code:
   .Retrieves the latest codebase from the repository.
   
3. Retrieve and Set Up Secrets:
   . Retrieve required secrets, such as API keys, from GitHub secrets.
   
4. Run Docker Containers:
.  Builds and runs your application's Docker containers using a specified docker-compose file.
.  Ensures the application is deployed in a consistent environment.
   
To enable the deploy job, uncomment the relevant lines and ensure the steps are configured correctly for your environment and application needs.

### Secrets and Environment Variables

`SONAR_TOKEN`: The authentication token for SonarQube analysis. Must be stored as a GitHub secret for security.
`SONAR_HOST_URL`: The URL of your SonarQube server or SonarCloud. Also stored as a GitHub secret.
For the deploy job:

To add a secret:

1. Go to your GitHub repository.
2. Navigate to Settings > Secrets and variables > Actions.
3. Click New repository secret.
4. Enter the name (`SONAR_TOKEN`, `SONAR_HOST_URL`, etc.) and the value.
   
## Running the Pipeline

To manually trigger this pipeline:
1. Commit any changes and push them to the main branch.
2. The pipeline (once on.push is uncommented) will automatically run for every push to main.
   
Alternatively, you can trigger the workflow from the Actions tab in your GitHub repository if you have enabled manual triggers or dispatch events.

### Troubleshooting
. SonarQube Issues:
       . Ensure the correct `SONAR_TOKEN` and `SONAR_HOST_URL` are configured as secrets.
   
     . Verify the sonar.projectKey matches the key defined in your SonarQube project.
   
. Docker Deployment Issues (for the deploy job):
      . Check that Docker is running on your self-hosted runner.
  
     . Ensure all required environment variables and secrets are properly configured.
  
     . Review `docker-compose.local.yml` for correct service definitions.
  
. Pipeline Failing:

     . Check the workflow logs in the Actions tab to see detailed error messages.

     . Make sure all paths and configurations reference the correct files and secrets.

### References
[GitHub Actions Documentation](https://docs.github.com/en/actions)

[SonarQube Documentation](https://docs.sonarsource.com/sonarqube/latest/devops-platform-integration/github-integration/importing-github-repositories/)

[GitHub Secrets Management](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions)

[Docker Documentation](https://docs.docker.com/)
