🔹 Step 1: Prepare the Slave Server

Login to Slave Server via SSH.

Install Java (required for Jenkins agent):

sudo apt update
sudo apt install openjdk-17-jdk -y
java -version

Install SSH Server (for Master → Slave communication):

sudo apt install openssh-server -y
sudo systemctl start ssh
sudo systemctl enable ssh
sudo systemctl status ssh

Create Jenkins User:

sudo adduser jenkins
sudo usermod -aG sudo jenkins

Check Slave IP (needed for Master):

ip a
# Example: 172.31.41.70
🔹 Step 2: Prepare the Master Server

Login to Master Server.

Install Java and Jenkins (if not already installed).

Generate SSH Key (for passwordless connection to Slave):

ssh-keygen -t ed25519
# Press Enter for all prompts

Files created: ~/.ssh/id_ed25519 (private key) and ~/.ssh/id_ed25519.pub (public key)

🔹 Step 3: Add Master Public Key to Slave

Switch to Jenkins user on Slave:

su - jenkins

Create .ssh folder and set permissions:

mkdir -p ~/.ssh
chmod 700 ~/.ssh

Add Master's public key:

nano ~/.ssh/authorized_keys
# Paste content of ~/.ssh/id_ed25519.pub from Master

Set correct permissions:

chmod 600 ~/.ssh/authorized_keys
🔹 Step 4: Test SSH Connection from Master

From Master server, run:

ssh jenkins@<slave-ip>

Expected Result:

Connects to slave without asking for a password

Exit with: exit

🔹 Step 5: Configure Slave in Jenkins UI

Login to Jenkins UI → http://<master-ip>:8080.

Create New Node:

Navigate: Manage Jenkins → Manage Nodes and Clouds → New Node

Name: slave1

Type: Permanent Agent → Click Create

Configure Node:

Remote root directory: /home/jenkins

Labels: slave

Usage: Use this node as much as possible

Launch method: Launch agents via SSH

Host: <slave-ip>

Add SSH Credentials:

Kind: SSH Username with private key

Username: jenkins

Private Key: Copy from Master

cat ~/.ssh/id_ed25519
# Paste in Jenkins → Add

Save Node:

Jenkins automatically connects

Copies agent.jar

Starts the agent

Status should show: Agent successfully connected

🔹 Step 6: Test Slave Agent

Create a Freestyle Project:

New Item → Freestyle Project

Restrict to run on slave:

Check Restrict where this project can run

Label Expression: slave

Add a build step:

hostname

Run the build → Output should show Slave hostname, confirming the job runs on the slave.

✅ All steps completed!

Master schedules jobs

Slave executes builds

Jenkins Agent setup is successful.
