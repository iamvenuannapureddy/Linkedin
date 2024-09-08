<h1>Project-11: Continuous Integration on Azure Cloud</h1>

**Project Overview**:
In this project, we’ll set up a Continuous Integration (CI) pipeline on Microsoft Azure. We will use Azure DevOps, which is Azure’s integrated platform for CI/CD. The pipeline will automatically build, test, and deploy a Java application to Azure App Service or Azure Kubernetes Service (AKS).

### **Project Architecture**:
- **Azure DevOps**: To orchestrate the CI pipeline.
- **Azure Repos (or GitHub)**: For source control and versioning.
- **Azure Pipelines**: For automating the build and testing process.
- **Azure App Service or Azure Kubernetes Service (AKS)**: For deploying the application.
- **Maven**: To build and package the Java application.
- **Docker**: To containerize the application if deploying to AKS.

---

### **Project Steps**:

#### **Step 1: Set Up Version Control (Azure Repos or GitHub)**
1. **Azure Repos**:
   - If you are using Azure Repos, create a repository under your Azure DevOps project and push your code to it.
   - If using GitHub, you can integrate GitHub with Azure Pipelines.
2. **Link Source Control to Azure DevOps**:
   - If using GitHub, link your GitHub repository to Azure DevOps by creating a new service connection.

---

#### **Step 2: Set Up Azure Pipelines for Continuous Integration**
1. **Create a Build Pipeline in Azure DevOps**:
   - In Azure DevOps, go to **Pipelines** and click **New Pipeline**.
   - Select the source repository (Azure Repos or GitHub).
   - Choose the type of pipeline: YAML for pipeline-as-code (preferred for version control).
   
2. **Define `azure-pipelines.yml`**:
   - Below is an example YAML file that defines the CI pipeline for a Java web application using Maven:

   ```yaml
   trigger:
     branches:
       include:
         - main

   pool:
     vmImage: 'ubuntu-latest'

   steps:
     - task: Maven@3
       inputs:
         mavenPomFile: 'pom.xml'
         goals: 'clean package'
         options: '-DskipTests'

     - task: CopyFiles@2
       inputs:
         SourceFolder: '$(System.DefaultWorkingDirectory)/target'
         Contents: '**/*.jar'
         TargetFolder: '$(Build.ArtifactStagingDirectory)'

     - task: PublishBuildArtifacts@1
       inputs:
         pathToPublish: '$(Build.ArtifactStagingDirectory)'
         artifactName: 'drop'
         publishLocation: 'Container'

     - task: Maven@3
       inputs:
         mavenPomFile: 'pom.xml'
         goals: 'test'
   ```

   **Explanation**:
   - **trigger**: The pipeline is triggered on every push to the `main` branch.
   - **pool**: Specifies the virtual machine image to use (`ubuntu-latest`).
   - **Maven@3**: This task runs Maven commands to build (`clean package`) and test the Java project.
   - **CopyFiles@2**: Copies the built JAR file to a staging directory.
   - **PublishBuildArtifacts@1**: Publishes the build artifacts (JAR file) to Azure DevOps, making it available for the deployment pipeline.

---

#### **Step 3: Set Up Continuous Testing**
1. **Include Unit Tests**:
   - In the `azure-pipelines.yml` file, add the following step after the `build` phase to run unit tests automatically after the build:

   ```yaml
   - task: Maven@3
     inputs:
       mavenPomFile: 'pom.xml'
       goals: 'test'
   ```

   This will ensure that every build passes the test suite before proceeding further.

---

#### **Step 4: Deploy to Azure (App Service or Kubernetes)**
1. **Option 1: Azure App Service**:
   - **Create an App Service**:
     - Go to the Azure portal and create an Azure App Service (select your runtime as Java or Docker if using containers).
   - **Set Up Azure DevOps Service Connection**:
     - In Azure DevOps, go to **Project Settings** > **Service connections**.
     - Create a new service connection for Azure Resource Manager, allowing Azure DevOps to deploy to Azure.
   - **Add Deployment Stage to Pipeline**:
     - Update the `azure-pipelines.yml` file to include deployment steps:

     ```yaml
     stages:
       - stage: Build
         jobs:
           - job: Build
             steps:
               ... # previous build steps

       - stage: Deploy
         jobs:
           - job: DeployAppService
             steps:
               - task: AzureWebApp@1
                 inputs:
                   azureSubscription: '<your-service-connection>'
                   appType: 'webApp'
                   appName: '<your-app-service-name>'
                   package: '$(Build.ArtifactStagingDirectory)/**/*.jar'
     ```

   **Explanation**:
   - **AzureWebApp@1**: Deploys the JAR file to the specified Azure App Service.

2. **Option 2: Deploy to Azure Kubernetes Service (AKS)**:
   - **Create AKS Cluster**:
     - In the Azure portal, create a Kubernetes cluster (AKS).
   - **Containerize the Application**:
     - Create a `Dockerfile` to containerize the Java application:

     ```dockerfile
     FROM openjdk:11-jre-slim
     COPY target/mywebapp.jar /usr/app/mywebapp.jar
     WORKDIR /usr/app
     EXPOSE 8080
     CMD ["java", "-jar", "mywebapp.jar"]
     ```

   - **Build and Push Docker Image to Azure Container Registry (ACR)**:
     - Use Azure Pipelines to build and push the Docker image to ACR:

     ```yaml
     - task: Docker@2
       inputs:
         containerRegistry: '<your-acr-service-connection>'
         repository: 'mywebapp'
         command: 'buildAndPush'
         dockerfile: '**/Dockerfile'
         tags: |
           $(Build.BuildId)
     ```

   - **Deploy to AKS**:
     - Use `kubectl` in Azure Pipelines to deploy the application to AKS:

     ```yaml
     - task: Kubernetes@1
       inputs:
         connectionType: 'Azure Resource Manager'
         azureSubscription: '<your-aks-connection>'
         azureResourceGroup: '<your-resource-group>'
         kubernetesCluster: '<your-aks-cluster>'
         namespace: 'default'
         command: 'apply'
         useConfigurationFile: true
         configuration: 'manifests/deployment.yml'
     ```

   **Explanation**:
   - **Docker@2**: Builds and pushes the Docker image to Azure Container Registry (ACR).
   - **Kubernetes@1**: Deploys the Docker image to AKS using the Kubernetes manifest files.

---

#### **Step 5: Monitoring and Notifications**
1. **Azure Monitor**:
   - Use Azure Monitor and Application Insights to track application performance and health metrics.
   - Set up alerts for deployment failures, high response times, or crashes.
2. **Notifications with Azure DevOps**:
   - Use Azure DevOps' built-in notifications or integrate with Slack to receive notifications when a build or deployment succeeds or fails.



