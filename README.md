# Optimized Jenkins Pipeline with Docker Agents
## Overview
This is a demonstration on how to optimize Jenkins pipelines using Docker container agents, allowing developers to isolate environments. It improves CI/CD efficiency, and ensures consistent builds across different stages. 
## What This Pipeline Does
This Jenkins pipeline is structured to run two separate stages—Back-end and Front-end—using Docker container agents to ensure isolated and consistent environments for each.
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

# Add pipeline code to Github
In Jenkins we will pull the pipeline code from Github.
In your repo create a new file called Jenkinsfile and add the following pipeline code to it.
```
pipeline {
  agent none
  stages {
    stage('Back-end') {
      agent {
        docker { image 'maven:3.8.1-adoptopenjdk-11' }
      }
      steps {
        sh 'mvn --version'
      }
    }
    stage('Front-end') {
      agent {
        docker { image 'node:16-alpine' }
      }
      steps {
        sh 'node --version'
      }
    }
  }
}
```
# Breakdown of the Pipeline
__agent none__
* This tells Jenkins not to use any default agent for the entire pipeline.
* Each stage will define its own agent, allowing greater flexibility and separation.
__Stage: Back-end__
* Uses a Docker image with Maven 3.8.1 and Java 11 (AdoptOpenJDK).
* Executes mvn --version inside the container to verify the Maven setup.
* Ideal for building/testing Java applications.
__Stage: Front-end__
* Uses a lightweight Docker image with Node.js 16 (Alpine-based).
* Runs node --version to check the Node.js environment.
* Perfect for front-end builds or JavaScript-based tools like npm or React.
# Create a New Pipeline Project
1. Click on “New Item” from the Jenkins dashboard.
2. Enter a name for your pipeline project (e.g., docker-agent-pipeline).
3. Select “Pipeline” as the project type.
4. Click “OK” to proceed to the pipeline configuration screen.
# Test Docker Agent Pipeline in Jenkins
Since our Jenkinsfile is committed to a GitHub repository,
1. In the __Pipeline__ section: Change __Definition__ to: __Pipeline script from SCM__
2. Set __SCM__ to: Git
3. Enter your Git Repository URL, for example:
```
https://github.com/GaganiR/jenkins-first-pipeline
```
4. If your repo is private: Add __Credentials__ for your GitHub (username/password or token)
5. Under __Script Path__, enter the relative path to your Jenkinsfile (default is just Jenkinsfile if it's at the root).
6. Click __Save__, then __Build Now__ to trigger the pipeline.
# View the Build Output
1. Click on the build number under __Build History__ (e.g., #1).
2. Click __Console Output__ to view:
* Docker images being pulled
* mvn --version result for back-end
* node --version result for front-end

You should see Jenkins automatically:
* Pull the required Docker images
* Run each stage in an isolated container
![console output 1](https://github.com/user-attachments/assets/b238697c-7e5d-4809-bfc9-bea9bc9c8b70)
![console output 3](https://github.com/user-attachments/assets/5d5801d0-b5bd-47d7-bd66-f104f782ddb6)
![console output 4](https://github.com/user-attachments/assets/4b075859-96cd-4971-a0f7-f3f2c43e8b6c)
# Explanaion on docker rm
The container used in each stage is __automatically removed__ after the stage runs.
This behavior is built into Jenkins’ use of docker { image '...' } agents.
It keeps the host environment clean and ensures isolated, reproducible builds.
This helps clean up temporary containers, preventing disk space buildup and resource leaks.

# See the processes being created and destroyed in real time
This automatic cleanup behavior can be observed in your terminal of the EC2 which has the Jenkins pipeline running.
To do this, after clicking on Build pipeline immediately keep entering the command ' docker ps' repeatedly.
![docker processes](https://github.com/user-attachments/assets/37af9fbb-e01f-4a2e-a978-9938d19c156b)
This ensures:
* No lingering containers
* No unused resources staying idle
* No manual cleanup required
# Contrast With Jenkins Nodes
On the other hand, Jenkins nodes (or static agents) are long-lived machines (VMs, EC2s, etc.) that:
* Stay online even when not in use.
* May accumulate old build artifacts, installed tools, or processes over time.
* Require manual maintenance to keep clean and updated.
# Final Thought
Using Docker agents makes Jenkins pipelines cleaner, safer, and more efficient by ensuring no unused containers or dependencies remain on the system after a job completes.
