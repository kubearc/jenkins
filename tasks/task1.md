Jenkins & Docker Practical Lab - Kubearc Academy

Welcome to the **Kubearc Academy** Jenkins Training Repository. This lab is designed to take you from a manual setup to a fully automated "Infrastructure as Code" pipeline.

##  Prerequisites
Before starting, ensure you have:
* **Jenkins** running on port `8080`.
* **Docker**/**Podman** installed and the Jenkins user added to the docker/podman group.
* **ngrok** running to expose your local Jenkins to GitHub.
* **GitHub Repo** This is the repo you will use in your Jenkins projects.


---

## Phase 1: Freestyle Projects & Multi-Triggering
**Objective:** Master the connectivity between GitHub and Jenkins.

### Task 1.1: The Webhook Integration
* Create a **Freestyle Project** named `Kubearc-Portal-Freestyle`.
* Configure the project to use this GitHub repository.
* **Trigger:** Select `GitHub hook trigger for GITScm polling`.
* **Action:** Update the `index.html` file, push it, and verify that Jenkins starts building automatically.

### Task 1.2: The "Triple Trigger" Configuration
Modify your project to support three different trigger types simultaneously:
1. **GitHub Webhook:** For real-time builds.
2. **Poll SCM:** Set the schedule to `H/5 * * * *` (Poll every 5 minutes).
3. **Build Periodically:** Set the schedule to `H 0 * * *` (A daily build at midnight).

---

##  Phase 2: Docker Multi-Stage & Custom HTTPD
**Objective:** Optimize production images and customize the web server.

### Task 2.1: Custom HTTPD Image
Create a `Dockerfile`/`Containerfile` that uses a custom Apache (`httpd`) image.
* **Requirement:** Use the `httpd:alpine` base for a smaller footprint.
* **Requirement:** Add a custom `LABEL` to the image: `LABEL maintainer="Kubearc-Academy-Student"`.

### Task 2.2: Multi-Stage Build Strategy
Write a `Dockerfile`/`Containerfile` that implements two stages:
1. **Stage 1 (Build):** Use an Alpine image to prepare/lint your code.
2. **Stage 2 (Final):** Use your custom `httpd` image. Copy only the verified code from Stage 1 into `/usr/local/apache2/htdocs/`.

---

## Phase 3: The Declarative Pipeline (Jenkinsfile)
**Objective:** Define the entire lifecycle as code.

### Task 3.1: The Master Pipeline
Create a file named `Jenkinsfile` in the root of your repo with the following stages:
1. **Checkout:** Pull code from GitHub.
2. **Linting:** Check for syntax errors.
3. **Build:** Build your custom Multi-Stage Docker image.
4. **Health Check:** - Run the container on port `8081`.
   - Use `curl` to verify the site returns a "200 OK".
   - Stop and remove the container.

---

##  Phase 4: Final Capstone Project
**Scenario:** Deploy the "Student Finance Portal" for Kubearc Academy.
* The pipeline must tag the image with the Jenkins Build Number (e.g., `kubearc-finance:v${BUILD_NUMBER}`).
* If the build fails, the `post` section must echo: "Deployment Failed - Rolling back to previous version".
* Successful builds must clean up all dangling Docker images to save space.

---
