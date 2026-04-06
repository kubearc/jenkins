# Install Jenkins with a volume in a container
```bash
podman volume create jenkins_home
podman run -d --name jenkins -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```
# Install jenkins in a RHEL system
```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/rpm-stable/jenkins.repo
sudo dnf upgrade
# Add required dependencies for the Jenkins package
sudo dnf install fontconfig java-21-openjdk
sudo dnf install jenkins
sudo systemctl daemon-reload
sudo systemctl enable jenkins --now
```
