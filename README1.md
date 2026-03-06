1️⃣ Jenkins Master Setup

Install Java and Jenkins:

sudo apt update
sudo apt install openjdk-17-jdk -y
sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins

Access Jenkins UI:

http://<master-ip>:8080

Unlock Jenkins using:

cat /var/lib/jenkins/secrets/initialAdminPassword

Install recommended plugins.

Create Jenkins Slave Node:

Manage Jenkins → Nodes → New Node

Name: slave

Remote root: /home/jenkins

Labels: java

Launch method: Launch agent via SSH

Credentials: Jenkins user credentials

Host/IP: slave private IP

Ensure Master can SSH into Slave.

2️⃣ Jenkins Slave Setup

Purpose: Execute builds, run Docker containers.

Create Jenkins user and give sudo:

sudo adduser jenkins
sudo usermod -aG sudo jenkins

Install Java, Git, Docker, Docker Compose:

sudo apt update
sudo apt install openjdk-17-jdk git -y
sudo apt install docker.io -y
sudo apt install docker-compose-v2 -y

Add Jenkins user to Docker group (so Docker commands can run without sudo):

sudo usermod -aG docker jenkins

Verify permissions:

sudo su - jenkins
docker ps
docker compose version

Restart Jenkins agent (so Jenkins picks up new group permissions):

sudo systemctl restart jenkins
3️⃣ GitHub Webhook Setup

Go to GitHub → Repository → Settings → Webhooks → Add webhook.

Fill in:

Field	Value
Payload URL:	http://<jenkins-master-ip>:8080/github-webhook/
Content type:	application/json
Secret:	(optional) leave blank
Events:	Just the push event

Click Add webhook.

Test by editing a file or pushing a commit to GitHub.

4️⃣ Jenkins Pipeline Code

Create a Pipeline job in Jenkins and use this script (or Jenkinsfile in your repo):

pipeline {
    agent { label 'java' }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Manju2023/product-spring-boot-project.git'
            }
        }

        stage('Stop Old Containers') {
            steps {
                sh 'docker compose down || true'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Run Containers') {
            steps {
                sh 'docker compose up -d'
            }
        }
    }

    post {
        success {
            echo 'Application deployed successfully'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}
5️⃣ Restart Jenkins (if needed)

On master:

sudo systemctl restart jenkins

