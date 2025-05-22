# Optimized Jenkins Pipeline with Docker Agents
## Overview
This is a demonstration on how to optimize Jenkins pipelines using Docker container agents, allowing developers to isolate environments. It improves CI/CD efficiency, and ensures consistent builds across different stages. 
## Prerequisites
1. Launch an AWS EC2 Instance of AMI: Ubuntu Server 22.04 LTS (or similar)
2. Configure Security Group to allow inbound traffic on port 8080. (Port 8080 is for Jenkins access since by default, Jenkins will not be accessible to the external world due to the inbound traffic restriction by AWS)
3. SSH into EC2 instance.
4. Install Java
```
sudo apt update
sudo apt install openjdk-17-jre
```
5. Verify Java is Installed
```
java -version
```
6. Install Jenkins
```
  curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```
# Login to Jenkins
Open your browser, type:
```
http://your-ec2-public-ip:8080
```
Then the screen willl prompt you to enter the admin password.
Get this by running the following command in your EC2 terminal.
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Copy the output and paste it on the jenkins login page.
Click on Install suggested plugins, create an admin user.
Proceed to Jenkins Dashboard.
# Install the Docker Pipeline plugin in Jenkins
* Go to Manage Jenkins > Manage Plugins.
* In the Available tab, search for "Docker Pipeline".
* Select the plugin and click the Install button.
* Restart Jenkins after the plugin is installed. Do this by adding /restart to the url of your jenkins page.
# Docker Agent Configuration
Install Docker
```
sudo apt update
sudo apt install docker.io
```
Add Jenkins User to Docker Group:
```
sudo usermod -aG docker jenkins
```
To test whether the docker agent configuration is successful, switch to jenkins user and run the following test command.
```
sudo su - jenkins
docker run hello-world
```
![hello docker](https://github.com/user-attachments/assets/c5fc57a4-eab3-48ea-9ca4-229b85b38ea7)

