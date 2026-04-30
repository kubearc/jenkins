# Jenkins Deployment to AWS EC2 via SSH

This guide provides a comprehensive, beginner-friendly walkthrough for setting up automated deployments between two AWS EC2 instances using the **Publish Over SSH** plugin.

---

##  1. Infrastructure Setup (AWS Console)

Before touching Jenkins, ensure your AWS instances can communicate.

### Security Group Configuration
1.  **Jenkins Master EC2**: Take note of its **Private IP**.
2.  **Target Server EC2**: 
    * Go to **Security Groups** -> **Edit Inbound Rules**.
    * Add a rule: **Type: SSH (Port 22)**.
    * **Source**: Select "Custom" and paste the **Private IP of the Jenkins Master**.
    * *This ensures only Jenkins can access the server via SSH.*

### Directory Permissions
On the **Target Server**, the web directory is usually protected. Run this to allow the deployment user (e.g., `ec2-user`) to upload files:
```bash
sudo chown -R ec2-user:ec2-user /var/www/html
```
2. SSH Key Handshake (The "Secret Password")
We use RSA keys to allow Jenkins to log in without a password.

Step A: Generate Keys on Jenkins Master
Connect to Jenkins EC2 via terminal.

Switch to the Jenkins user:

```Bash
sudo su - jenkins
Generate the key pair:
```
```Bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
View and Copy the Public Key:
```
```Bash
cat ~/.ssh/id_rsa.pub
Step B: Authorize Key on Target Server
Connect to Target EC2.
```
Open the authorized keys file:
```
Bash
nano ~/.ssh/authorized_keys
Paste the Public Key string from Step A on a new line at the bottom.
```
Save and Exit (Ctrl+O, Enter, Ctrl+X).

⚙️ 3. Jenkins Global Configuration
Install Plugin: Manage Jenkins > Plugins > Available > Search Publish Over SSH > Install.

Add Private Key:

On Jenkins Master, run cat ~/.ssh/id_rsa and copy the entire block.
```
Go to Manage Jenkins -> System.

Find Publish over SSH.

Paste the private key into the Key box.

Add SSH Server:

Name: AWS_Target_Server

Hostname: Private IP of Target EC2.

Username: ec2-user (or ubuntu).

Remote Directory: /var/www/html

Test Configuration: Click the button. It should return Success.
```
🛠️ 4. Method A: Freestyle Project (UI-Based)
Create New Item -> Freestyle Project.

Source Code Management: Select Git and enter your Repository URL.

Post-build Actions:

Click Add post-build action -> Send build artifacts over SSH.

SSH Server: Select AWS_Target_Server.

Source files: index.html (or /* for all files).

Exec command:
```
Bash
sudo systemctl restart httpd
```
Save and Build Now.


💻 5. Method B: Pipeline Project (Jenkinsfile)
Create a file named Jenkinsfile in your Git repository root:
```
pipeline {
    agent any

    environment {
        // Must match the name in Jenkins Global System Config
        SSH_SERVER_NAME = 'AWS_Target_Server'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Pre-requisites Check') {
            steps {
                script {
                    echo "Checking if artifacts exist..."
                    if (!fileExists('index.html')) {
                        error "Deployment aborted: index.html not found!"
                    }
                }
            }
        }

        stage('Deploy via SSH') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: "${SSH_SERVER_NAME}",
                            transfers: [
                                sshTransfer(
                                    sourceFiles: 'index.html',
                                    remoteDirectory: '/', // Relative to the path in Global Config
                                    execCommand: 'sudo systemctl restart httpd'
                                )
                            ],
                            verbose: true
                        )
                    ]
                )
            }
        }
    }
}
```
