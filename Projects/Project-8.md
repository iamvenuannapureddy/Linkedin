<h1>Project-8: Continuous Integration with Jenkins, Nexus, SonarQube, and Slack</h1>

**Project Overview**:
In this project, we’ll set up a Continuous Integration (CI) pipeline using Jenkins, Nexus, SonarQube, and Slack. Jenkins will orchestrate the CI process, Nexus will manage artifacts, SonarQube will handle static code analysis, and Slack will notify the team of the build results.

### **Project Steps**:

1. **Setup Jenkins**:
   - Install Jenkins on your server or local machine.
   - Install necessary plugins:
     - **Git**: For source control management.
     - **Maven Integration**: If using Maven for building the project.
     - **SonarQube Scanner**: For integrating SonarQube analysis.
     - **Nexus Artifact Uploader**: For uploading artifacts to Nexus.
     - **Slack Notification**: For sending notifications to Slack.

2. **Setup Nexus Repository**:
   - Install Nexus Repository Manager (OSS version is sufficient).
   - Create a Maven repository (hosted) to store your build artifacts.
   - Note the repository URL and configure Nexus credentials in Jenkins (Manage Jenkins > Credentials).

3. **Setup SonarQube**:
   - Install SonarQube on your server or use a cloud-hosted version.
   - Install SonarQube Scanner in Jenkins (Manage Jenkins > Global Tool Configuration).
   - Create a new SonarQube project and note the project key and authentication token.
   - Configure SonarQube server details in Jenkins (Manage Jenkins > Configure System).

4. **Setup Slack for Notifications**:
   - Create a Slack App and add it to your workspace.
   - Obtain the Slack incoming webhook URL.
   - Install and configure the Slack Notification plugin in Jenkins (Manage Jenkins > Configure System > Global Slack Notifier Settings).
   - Add the Slack channel, token, and webhook URL.

5. **Create a Jenkins Pipeline**:
   - Define your pipeline in a `Jenkinsfile`. Below is an example:

   ```groovy
   pipeline {
       agent any

       environment {
           // Set up environment variables
           MAVEN_HOME = tool 'Maven 3.6.3'
           SONARQUBE_URL = 'http://your-sonarqube-server'
           NEXUS_REPO = 'http://your-nexus-server/repository/maven-releases'
           NEXUS_CREDENTIALS_ID = 'nexus-credentials'
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
                   sh "${MAVEN_HOME}/bin/mvn clean package"
               }
           }

           stage('SonarQube Analysis') {
               steps {
                   // Run SonarQube analysis
                   withSonarQubeEnv('SonarQube Server') {
                       sh "${MAVEN_HOME}/bin/mvn sonar:sonar -Dsonar.projectKey=myproject"
                   }
               }
           }

           stage('Upload to Nexus') {
               steps {
                   // Upload artifact to Nexus
                   script {
                       def pom = readMavenPom file: 'pom.xml'
                       def artifactId = pom.artifactId
                       def version = pom.version

                       nexusArtifactUploader artifacts: [[artifactId: artifactId,
                                                         file: "target/${artifactId}-${version}.jar",
                                                         type: 'jar']],
                                             credentialsId: NEXUS_CREDENTIALS_ID,
                                             groupId: pom.groupId,
                                             nexusUrl: NEXUS_REPO,
                                             repository: 'maven-releases',
                                             version: version
                   }
               }
           }
       }

       post {
           success {
               slackSend channel: '#ci-cd', message: "Build successful: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
           }
           failure {
               slackSend channel: '#ci-cd', message: "Build failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
           }
       }
   }
   ```

   **Explanation**:
   - **pipeline**: Defines the pipeline block, which includes all pipeline configurations.
   - **agent any**: Jenkins will run the pipeline on any available agent.
   - **environment**: Sets up environment variables used in the pipeline.
   - **stages**: 
     - **Checkout**: Pulls the code from a Git repository.
     - **Build**: Compiles the Java application using Maven.
     - **SonarQube Analysis**: Runs static code analysis using SonarQube.
     - **Upload to Nexus**: Uploads the built artifact (JAR file) to Nexus.
   - **post**: Handles actions that should occur after the pipeline completes:
     - **success**: Sends a success notification to Slack.
     - **failure**: Sends a failure notification to Slack.

6. **Configure Jenkins Job**:
   - Create a new pipeline job in Jenkins.
   - In the job configuration, point to your Git repository, and specify the path to the `Jenkinsfile`.

7. **Run the Pipeline**:
   - Trigger the pipeline manually or set up a trigger (e.g., GitHub webhook) for automatic builds.
   - Jenkins will execute the steps defined in the `Jenkinsfile`, performing code checkout, building the project, running SonarQube analysis, uploading artifacts to Nexus, and sending Slack notifications.

8. **Monitor and Review**:
   - **SonarQube**: Review the static code analysis results in the SonarQube dashboard.
   - **Nexus**: Verify the artifact is uploaded correctly in the Nexus repository.
   - **Slack**: Check Slack for build notifications.

