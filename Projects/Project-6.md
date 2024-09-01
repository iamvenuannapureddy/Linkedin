<h1>Project-6: Containerization of a Java Project Using Docker</h1>

**Project Overview**:
Containerizing a Java application using Docker allows the application to be run consistently across different environments, from development to production. This project demonstrates how to containerize a Java application, specifically a Spring Boot application, using Docker. The goal is to create a Docker image that encapsulates the entire application, including its runtime environment, dependencies, and configuration.

**Steps to Containerize a Java Project Using Docker**:

1. **Prerequisites**:
   - **Java Development Kit (JDK)** installed on your local machine.
   - **Java project** (for example, a Spring Boot application).
   - **Maven** or **Gradle** for building the Java project.
   - **Docker** installed on your local machine.

2. **Create a Dockerfile**:
   The Dockerfile is a script that contains a series of commands to assemble the Docker image.

   ```Dockerfile
   # Use an official Java runtime as a parent image
   FROM openjdk:17-jdk-alpine

   # Set the working directory in the container
   WORKDIR /app

   # Copy the application's jar file to the container
   COPY target/myapp.jar /app/myapp.jar

   # Make port 8080 available to the world outside this container
   EXPOSE 8080

   # Run the jar file
   ENTRYPOINT ["java","-jar","/app/myapp.jar"]
   ```

   **Explanation**:
   - **FROM**: Specifies the base image. `openjdk:17-jdk-alpine` is a lightweight JDK image based on Alpine Linux.
   - **WORKDIR**: Sets the working directory inside the container.
   - **COPY**: Copies the JAR file from the target directory of your Java project to the container.
   - **EXPOSE**: Exposes port 8080, which is the default port for Spring Boot applications.
   - **ENTRYPOINT**: Specifies the command to run the application when the container starts.

3. **Build the Java Project**:
   Build your Java project using Maven or Gradle to generate the executable JAR file.

   ```bash
   mvn clean package
   ```

   This command will generate a `myapp.jar` file in the `target` directory.

4. **Build the Docker Image**:
   Use the Dockerfile to build a Docker image.

   ```bash
   docker build -t myapp:1.0 .
   ```

   **Explanation**:
   - **docker build**: Command to build a Docker image.
   - **-t myapp:1.0**: Tags the image with a name (`myapp`) and a version (`1.0`).
   - **.**: Specifies the current directory as the build context.

5. **Run the Docker Container**:
   After the image is built, you can run a container from the image.

   ```bash
   docker run -p 8080:8080 myapp:1.0
   ```

   **Explanation**:
   - **docker run**: Command to run a Docker container.
   - **-p 8080:8080**: Maps port 8080 of the container to port 8080 on your local machine.
   - **myapp:1.0**: Specifies the Docker image to run.

6. **Test the Application**:
   Once the container is running, you can access the application by navigating to `http://localhost:8080` in your web browser. If your application has a REST API, you can use tools like Postman or `curl` to test the endpoints.

7. **Push the Docker Image to a Registry (Optional)**:
   If you want to share your Docker image, you can push it to a Docker registry like Docker Hub.

   ```bash
   docker tag myapp:1.0 mydockerhubusername/myapp:1.0
   docker push mydockerhubusername/myapp:1.0
   ```

   **Explanation**:
   - **docker tag**: Tags your image for the Docker registry.
   - **docker push**: Pushes the image to the specified Docker Hub repository.

8. **Automate the Process with a CI/CD Pipeline (Optional)**:
   To automate the building, testing, and deployment of your Docker container, integrate Docker with your CI/CD pipeline (e.g., Jenkins, GitHub Actions, GitLab CI). This allows you to automatically build and deploy new images whenever you commit changes to your codebase.


