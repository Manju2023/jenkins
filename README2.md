steps for ansible and jenkins configuration along dockerized application
🏗 Final Architecture
Developer → GitHub
        ↓
Jenkins (Master Server)
        ↓
Ansible
        ↓
Slave Server
        ↓
Docker Container (Spring Boot App)

Master: Jenkins + Ansible
Slave: Docker host where containers run

1️⃣ Master Server Initial Setup

Login to master server.

Update system:

sudo apt update
sudo apt upgrade -y
2️⃣ Create Ansible User on Master
sudo adduser ansible

Give sudo permission:

sudo usermod -aG sudo ansible

Switch to ansible:

su - ansible
3️⃣ Install Ansible (Master)
sudo apt install ansible -y

Verify:

ansible --version
4️⃣ Install Java (Required for Jenkins)

Switch back to root or ubuntu user.

sudo apt install openjdk-17-jdk -y

Check:

java -version
5️⃣ Install Jenkins

Add Jenkins repository:

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

Install Jenkins:

sudo apt update
sudo apt install jenkins -y

Start Jenkins:

sudo systemctl start jenkins
sudo systemctl enable jenkins
6️⃣ Access Jenkins

Open browser:

http://MASTER_IP:8080

Get admin password:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Install Suggested Plugins.

7️⃣ Create DevOps Project Folder

Switch back to ansible user:

su - ansible

Create folder:

mkdir devops-project
cd devops-project

Structure:

devops-project
│
├── inventory.ini
└── setup-slave.yml
8️⃣ Generate SSH Key (Master)

Run:

ssh-keygen

Press Enter for all prompts.

Keys created:

/home/ansible/.ssh/id_rsa
/home/ansible/.ssh/id_rsa.pub
9️⃣ Slave Server Setup

Login to slave server.

Create ansible user:

sudo adduser ansible

Give sudo permission:

sudo usermod -aG sudo ansible
🔟 Copy Public Key to Slave (for ansible)

On master:

cat ~/.ssh/id_rsa.pub

Copy the output.

Login to slave:

sudo su - ansible

Create SSH directory:

mkdir -p ~/.ssh
chmod 700 ~/.ssh

Add key:

nano ~/.ssh/authorized_keys

Paste the public key.

Set permission:

chmod 600 ~/.ssh/authorized_keys
1️⃣1️⃣ Create Jenkins User on Slave
sudo adduser jenkins

Add Docker group later via playbook.

Add SSH key for Jenkins:

sudo su - jenkins
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys

Paste the same public key.

Set permission:

chmod 600 ~/.ssh/authorized_keys
1️⃣2️⃣ Test SSH from Master

From master:

Test ansible connection:

ssh ansible@SLAVE_IP

Test jenkins connection:

ssh jenkins@SLAVE_IP

Both should login without password.

1️⃣3️⃣ Create Inventory File

On master:

nano inventory.ini

Add:

[dev]
SLAVE_IP ansible_user=ansible

Example:

[dev]
192.168.1.20 ansible_user=ansible

Meaning:

Ansible connects to slave using ansible user
1️⃣4️⃣ Test Ansible Connection
ansible -i inventory.ini dev -m ping

Expected:

SUCCESS
1️⃣5️⃣ Create Ansible Playbook

Create file:

nano setup-slave.yml

Playbook:

---
- name: Configure Docker Slave
  hosts: dev
  become: yes

  tasks:

  - name: Update packages
    apt:
      update_cache: yes

  - name: Install Docker and Git
    apt:
      name:
        - docker.io
        - git
      state: present

  - name: Start Docker service
    service:
      name: docker
      state: started
      enabled: yes

  - name: Add Jenkins user to Docker group
    user:
      name: jenkins
      groups: docker
      append: yes
1️⃣6️⃣ Run Playbook
ansible-playbook -i inventory.ini setup-slave.yml

This installs:

Docker

Git

Docker permissions for Jenkins

1️⃣7️⃣ Verify Docker on Slave

Login to slave:

ssh jenkins@SLAVE_IP

Run:

docker ps

If it works without permission error → correct.

1️⃣8️⃣ Configure Jenkins Agent

In Jenkins:

Manage Jenkins
→ Nodes
→ New Node

Create node:

Name: docker-slave
Type: Permanent Agent

Configuration:

Remote root directory:

/home/jenkins

Labels:

docker

Launch method:

Launch agent via SSH

Host:

SLAVE_IP
1️⃣9️⃣ Add Jenkins SSH Credential
Manage Jenkins
→ Credentials
→ Global
→ Add Credentials

Type:

SSH Username with private key

Username:

jenkins

Private key:

Paste content of:

cat /home/ansible/.ssh/id_rsa
2️⃣0️⃣ Create Jenkins Pipeline

Create new pipeline job.

Pipeline script:

pipeline {

    agent { label 'docker' }

    stages {

        stage('Clone Repository') {
            steps {
                git 'https://github.com/Manju2023/product-spring-boot-project.git'
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
}
2️⃣1️⃣ GitHub Repo Must Contain
Dockerfile
docker-compose.yml
Spring Boot project

Example docker-compose:

services:
  app:
    build: .
    ports:
      - "8081:8080"
2️⃣2️⃣ Run Pipeline

Click:

Build Now

Pipeline flow:

Clone Repo
↓
Stop Containers
↓
Build Image
↓
Run Container
🎯 Final Result

Application runs at:

http://SLAVE_IP:8081
