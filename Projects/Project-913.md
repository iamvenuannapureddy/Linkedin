<h1>Project-13: GitHub Actions for CI/CD</h1>

**Project Overview**:
In this project, we'll set up a CI/CD pipeline using **GitHub Actions** for a Java application. GitHub Actions is a powerful tool for automating software workflows directly within GitHub. We will build, test, and deploy the application using GitHub Actions, making use of both continuous integration (CI) and continuous delivery (CD) practices.

### **Project Architecture**:
- **GitHub Actions**: To define and automate the CI/CD workflow.
- **Maven**: For building the Java application.
- **Docker** (optional): To containerize the application, if required.
- **AWS Elastic Beanstalk / ECS / Azure App Service**: For deploying the application.
- **GitHub Secrets**: For storing sensitive credentials (AWS or Azure keys).
- **Slack/Email Notifications**: For receiving notifications on the status of builds and deployments.

---

### **Project Steps**:

#### **Step 1: Set Up GitHub Repository**
1. **Create or Use Existing GitHub Repository**:
   - Create a new GitHub repository (or use an existing one) for your Java application.
   - Push your Java application code, including the `pom.xml` file if you are using Maven as the build tool.

---

#### **Step 2: Define GitHub Actions Workflow**
1. **Create a GitHub Actions Workflow File**:
   - In the root of your repository, create the directory `.github/workflows/`.
   - Inside the `workflows` directory, create a new file called `ci-cd.yml` or any other name you prefer.

2. **Example CI/CD Workflow File (`ci-cd.yml`)**:

   ```yaml
   name: CI/CD Pipeline

   on:
     push:
       branches:
         - main
     pull_request:
       branches:
         - main

   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - name: Checkout code
           uses: actions/checkout@v2

         - name: Set up JDK 11
           uses: actions/setup-java@v2
           with:
             java-version: '11'

         - name: Cache Maven packages
           uses: actions/cache@v2
           with:
             path: ~/.m2
             key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
             restore-keys: |
               ${{ runner.os }}-maven-

         - name: Build with Maven
           run: mvn clean install -DskipTests

         - name: Run unit tests
           run: mvn test

     deploy:
       needs: build
       runs-on: ubuntu-latest
       steps:
         - name: Checkout code
           uses: actions/checkout@v2

         - name: Deploy to AWS Elastic Beanstalk
           env:
             AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
             AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
             AWS_REGION: 'us-east-1'
           run: |
             zip -r app.zip .
             aws elasticbeanstalk create-application-version \
               --application-name MyApp \
               --version-label v1 \
               --source-bundle S3Bucket=my-bucket,S3Key=app.zip
             aws elasticbeanstalk update-environment \
               --environment-name MyApp-env \
               --version-label v1
   ```

   **Explanation**:
   - **on: push**: Triggers the pipeline on pushes to the `main` branch or when a pull request is made to `main`.
   - **jobs: build**:
     - The `build` job runs on `ubuntu-latest`.
     - It checks out the code, sets up JDK 11, caches Maven dependencies, and runs `mvn clean install` to build the Java project.
     - It also runs unit tests to ensure code quality.
   - **jobs: deploy**:
     - This job depends on the `build` job and will only run if the build job is successful.
     - It uses AWS CLI to deploy the application to **AWS Elastic Beanstalk**.

---

#### **Step 3: Set Up Secrets in GitHub**
1. **Add Secrets for AWS or Other Providers**:
   - Go to your GitHub repository’s **Settings** > **Secrets and variables** > **Actions** > **New repository secret**.
   - Add the following secrets (for AWS deployment):
     - `AWS_ACCESS_KEY_ID`
     - `AWS_SECRET_ACCESS_KEY`

2. **For Azure** (if using Azure App Service):
   - Add Azure credentials as secrets, such as:
     - `AZURE_APP_ID`
     - `AZURE_PASSWORD`
     - `AZURE_TENANT`

---

#### **Step 4: Deploy to AWS Elastic Beanstalk (or ECS)**

1. **Option 1: AWS Elastic Beanstalk**:
   - The workflow deploys the application to Elastic Beanstalk using the AWS CLI. Ensure that the application is zipped and uploaded to an S3 bucket, and then create a new application version.
   - **Elastic Beanstalk Configuration**:
     - Create an Elastic Beanstalk application in the AWS console.
     - Set up an environment with the correct platform (Java SE).

2. **Option 2: AWS ECS (for Dockerized Applications)**:
   - If you are using ECS, modify the deployment step to build and push the Docker image to **Amazon ECR** and deploy to an ECS cluster.

   Update the `deploy` step in the `ci-cd.yml` file to build the Docker image and push it to Amazon ECR:

   ```yaml
   deploy:
     needs: build
     runs-on: ubuntu-latest
     steps:
       - name: Checkout code
         uses: actions/checkout@v2

       - name: Log in to Amazon ECR
         run: |
           aws ecr get-login-password --region ${{ secrets.AWS_REGION }} \
             | docker login --username AWS --password-stdin ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com

       - name: Build Docker image
         run: docker build -t my-app .

       - name: Tag Docker image
         run: docker tag my-app:latest ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com/my-app:latest

       - name: Push Docker image to Amazon ECR
         run: docker push ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.${{ secrets.AWS_REGION }}.amazonaws.com/my-app:latest

       - name: Deploy to Amazon ECS
         run: aws ecs update-service --cluster my-cluster --service my-service --force-new-deployment
   ```

   This workflow builds the Docker image, tags it, pushes it to ECR, and then deploys it to ECS.

---

#### **Step 5: Slack or Email Notifications (Optional)**
1. **Add Slack Notifications**:
   - You can configure Slack notifications for build and deployment events. Add a Slack bot or webhook URL to send notifications.
   - Example step for Slack:

   ```yaml
   - name: Send notification to Slack
     uses: 8398a7/action-slack@v3
     with:
       status: always
       fields: repo,message,commit,author
     env:
       SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
       SLACK_CHANNEL: 'deployments'
   ```

2. **Add Email Notifications**:
   - Use an email service to send notifications for build status. This can be configured using GitHub Actions Marketplace actions like **action-send-mail**.

---

#### **Step 6: Monitor and Debug Pipeline**
1. **GitHub Actions Logs**:
   - You can view detailed logs of your workflow by going to the **Actions** tab in your GitHub repository. Each job step will have corresponding logs that provide insights into failures or successes.
   
2. **GitHub Notifications**:
   - GitHub will automatically notify you of build and deployment statuses, but you can further customize notifications using Slack or other services as mentioned
