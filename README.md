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
## Installing On Amazon Linux

## Create a script which contains the installations commands names **jenkins-install.sh** file 
```bash 
vim jenkins-install.sh
```
Put below in the file
```
wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
 rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum upgrade
# Add required dependencies for the jenkins package
yum install fontconfig java-21-amazon-corretto -y
yum install jenkins -y
systemctl daemon-reload
systemctl enable --now jenkins.service

```
## Now change the permission of the file 
```bash 
chmod +x jenkins-install.sh
```
## Run the script 
```bash 
./jenkins-install.sh
```

## Verify the Installaion
```bash 
jenkins --version
```
