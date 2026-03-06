🔷 Jenkins Master–Slave Architecture
MASTER SERVER (Jenkins Installed)
        │
        │  SSH Connection
        ▼
SLAVE SERVER (Runs Builds)

Master schedules jobs → Slave executes jobs
🖥️ 1️⃣ Steps in SLAVE SERVER

Login to the slave machine.

Step 1: Install Java

Jenkins agents require Java.

sudo apt update
sudo apt install openjdk-17-jdk -y

Check:

java -version
Step 2: Install SSH Server
sudo apt install openssh-server -y

Start SSH:

sudo systemctl start ssh
sudo systemctl enable ssh

Check status:

sudo systemctl status ssh
Step 3: Create Jenkins User
sudo adduser jenkins

Give sudo permission:

sudo usermod -aG sudo jenkins
Step 4: Check Slave IP
ip a

Example:

172.31.41.70(private ip of slave)

Save this IP.

🖥️ 2️⃣ Steps in MASTER SERVER

Login to the master machine (Jenkins installed).

install java and jenkins

Step 1: Generate SSH Key
ssh-keygen -t ed25519

Press Enter for all questions.

Files created:

~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
🔑 3️⃣ Add Master Public Key to Slave



1️⃣ Switch to Jenkins User
su - jenkins

Enter the jenkins password.

You should see:

jenkins@slave:~$
3️⃣ Create .ssh Folder
mkdir -p ~/.ssh
chmod 700 ~/.ssh
4️⃣ Add the Public Key
nano ~/.ssh/authorized_keys

Paste the public key from master:

ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKBKlLDvlx9mo25MdaEHEGaTKFyjvKWT7/0/X8QtkVSz ubuntu@master

Save:

CTRL + X
Y
ENTER

5️⃣ Set Correct Permissions
chmod 600 ~/.ssh/authorized_keys
4️⃣ Test Connection From MASTER

Go back to Master server:

ssh jenkins@172.31.41.70

Expected result:

jenkins@slave:~$

No password should be asked.

Exit:

exit
🌐 5️⃣ Steps in JENKINS UI

Open Jenkins:

http://<MASTER-IP>:8080
Step 1: Create New Node

Navigate:

Manage Jenkins
      ↓
Manage Nodes and Clouds
      ↓
New Node

Enter:

Name: slave1
Type: Permanent Agent

Click Create.

Step 2: Configure Node
Remote root directory: /home/jenkins
Labels: slave
Usage: Use this node as much as possible
Launch method: Launch agents via SSH
Host: 172.31.41.70(private ip of slave)
Step 3: Add Credentials

Click Add Credentials

Kind: SSH Username with private key
Username: jenkins
Private Key: Enter directly

On Master server run:

cat ~/.ssh/id_ed25519

Copy everything:

-----BEGIN OPENSSH PRIVATE KEY-----
....
-----END OPENSSH PRIVATE KEY-----

Paste in Jenkins → Add.

Step 4: Save Node

Click Save.

Jenkins will automatically:

1️⃣ Connect to slave using SSH
2️⃣ Copy agent.jar
3️⃣ Start the Jenkins agent

You will see:

Agent successfully connected
🧪 6️⃣ Test the Slave Agent

Create job.

New Item
   ↓
Freestyle Project

Enable:

Restrict where this project can run
Label Expression: slave

Add build step:

hostname

Run build.

Output should show:

slave
✅ Final Working Flow
SLAVE SERVER
Install Java
Install SSH
Create jenkins user
Get IP address
        ▲
        │
MASTER SERVER
ssh-keygen
Add public key to slave
ssh-copy-id jenkins@slave-ip
Test ssh connection
        ▲
        │
JENKINS UI
Create node
Add SSH credentials
Connect agent
        ▲
        │
BUILD JOB RUNS ON SLAVE