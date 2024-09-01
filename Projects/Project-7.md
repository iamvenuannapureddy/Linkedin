<h1>Projects-7 Jenkins Pipeline as Code (Groovy) Project</h1>

**Project Overview**:
Jenkins Pipeline as Code allows you to define the entire continuous integration/continuous delivery (CI/CD) pipeline in a `Jenkinsfile`, which is a Groovy-based script. This approach improves transparency, version control, and reproducibility of the CI/CD pipeline. In this project, we’ll demonstrate how to create a Jenkins Pipeline as Code for building, testing, and deploying a Java application using Jenkins and Groovy.

### **Project Steps**:

1. **Setup Jenkins**:
   - Install Jenkins on your server or local machine.
   - Install necessary plugins: 
     - **Pipeline** (for pipeline jobs)
     - **Git** (for source control)
     - **Docker** (if building Docker images)
     - **Maven Integration** (if using Maven)

2. **Create a Java Project**:
   - For this example, we will assume you have a Java project, possibly a Spring Boot application, managed by Maven or Gradle.

3. **Create a Jenkinsfile**:
   - The `Jenkinsfile` is where you define your pipeline. Here is a basic example:

   ```groovy
   pipeline {
       agent any

       environment {
           // Set up any environment variables
           DOCKER_IMAGE = 'mydockerhubusername/myapp'
       }

       stages {
           stage('Checkout') {
               steps {
                   // Checkout code from Git repository
                   git url: 'https://github.com/username/repository.git', branch: 'main'
               }
           }

           stage('Build') {
               steps {
                   // Build the project using Maven
                   sh 'mvn clean package'
               }
           }

           stage('Test') {
               steps {
                   // Run tests
                   sh 'mvn test'
               }
           }

           stage('Build Docker Image') {
               steps {
                   // Build Docker image
                   script {
                       def appVersion = readMavenPom().getVersion()
                       docker.build("${DOCKER_IMAGE}:${appVersion}")
                   }
               }
           }

           stage('Push Docker Image') {
               steps {
                   // Push Docker image to Docker Hub
                   script {
                       def appVersion = readMavenPom().getVersion()
                       docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                           docker.image("${DOCKER_IMAGE}:${appVersion}").push()
                       }
                   }
               }
           }

           stage('Deploy') {
               steps {
                   // Deploy to a server or Kubernetes cluster
                   // For example, using kubectl
                   sh '''
                   kubectl set image deployment/myapp-deployment myapp=${DOCKER_IMAGE}:${appVersion}
                   '''
               }
           }
       }

       post {
           always {
               // Cleanup or notifications
               cleanWs()
           }
           success {
               echo 'Pipeline succeeded!'
           }
           failure {
               echo 'Pipeline failed!'
           }
       }
   }
   ```

   **Explanation**:
   - **pipeline**: Defines the pipeline block which includes all pipeline configurations.
   - **agent any**: Jenkins will run the pipeline on any available agent.
   - **environment**: Defines environment variables that can be used in the pipeline.
   - **stages**: Each stage in the pipeline represents a step in the CI/CD process.
   - **Checkout**: Fetches the code from a Git repository.
   - **Build**: Compiles the Java application using Maven.
   - **Test**: Runs the tests defined in the project.
   - **Build Docker Image**: Uses Docker to build an image based on the Maven-built application JAR.
   - **Push Docker Image**: Pushes the Docker image to Docker Hub.
   - **Deploy**: Deploys the application using Kubernetes, or any other deployment method.
   - **post**: Defines actions to perform after the pipeline completes, regardless of success or failure.

4. **Configure Jenkins Job**:
   - Create a new pipeline job in Jenkins.
   - In the job configuration:
     - Point to your Git repository.
     - Specify the path to the `Jenkinsfile` (if it's in the root, you can leave it as `Jenkinsfile`).

5. **Run the Pipeline**:
   - After configuring, you can manually trigger the job to run the pipeline.
   - Jenkins will execute the steps defined in your `Jenkinsfile`, providing real-time feedback.

6. **Pipeline Stages Breakdown**:

   - **Checkout Stage**: 
     - This stage pulls the code from the Git repository.
   
   - **Build Stage**: 
     - Uses Maven to compile the Java project and package it into a JAR file.
   
   - **Test Stage**: 
     - Runs all the unit tests, ensuring code quality before proceeding.
   
   - **Build Docker Image Stage**: 
     - Reads the version from `pom.xml` and builds a Docker image, tagging it with the application version.
   
   - **Push Docker Image Stage**: 
     - Pushes the Docker image to a Docker Hub repository. It uses Docker credentials stored in Jenkins.
   
   - **Deploy Stage**: 
     - Updates a Kubernetes deployment to use the new Docker image. This can be modified to use other deployment strategies (e.g., using `scp` for a server).

7. **Post Actions**:
   - **always**: Runs actions that should occur regardless of the pipeline result, such as workspace cleanup.
   - **success**: Runs when the pipeline completes successfully.
   - **failure**: Runs when the pipeline fails.
