## Netflix clone website

DevSecOps : Netflix Clone CI-CD with Monitoring | Email


Project Steps :-
Step 1 — Launch an Ubuntu(22.04) T2 Large Instance
Step 2 — Install Jenkins, Docker and Trivy. Create a Sonarqube Container using Docker.
Step 3 — Create a TMDB API Key.
Step 4 — Install Prometheus and Grafana On the new Server.
Step 5 — Install the Prometheus Plugin and Integrate it with the Prometheus server.
Step 6 — Email Integration With Jenkins and Plugin setup.
Step 7 — Install Plugins like JDK, Sonarqube Scanner, Nodejs, and OWASP Dependency Check.
Step 8 — Create a Pipeline Project in Jenkins using a Declarative Pipeline
Step 9 — Install OWASP Dependency Check Plugins
Step 10 — Docker Image Build and Push
Step 11 — Deploy the image using Docker
Step 12 — Kubernetes master and slave setup on Ubuntu (20.04)
Step 13 — Access the Netflix app on the Browser.
Step 14 — Terminate the AWS EC2 Instances.


Step 1: 
Launch an Ubuntu(22.04) T2 Large Instance
Launch an AWS T2 Large Instance. Use the image as Ubuntu. You can create a new key pair or use an existing one. Enable HTTP and HTTPS settings in the Security Group and open all ports (not best case to open all ports but just for learning purposes it's okay).


ssh -i <path-to-your-key>.pem ubuntu@<public-ip-of-instance>


Step 2:
## Install Jenkins
sudo vi jenkins.sh

#!/bin/bash
sudo apt-get update -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt-get update -y
sudo apt-get install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins

Make the script executable and run it:

sudo chmod 777 jenkins.sh
./jenkins.sh


## Install Docker
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER  
newgrp docker
sudo chmod 777 /var/run/docker.sock


## Install Trivy
sudo vi trivy.sh

#!/bin/bash
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y

Make the script executable and run it:
sudo chmod 777 trivy.sh
./trivy.sh

Create SonarQube Container
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

http://<EC2-Public-IP>:9000

Step 3: 
Create a TMDB API Key
Next, we will create a TMDB API key
Open a new tab in the Browser and search for TMDB
Click on the first result, you will see this page
Click on the Login on the top right. You will get this page.
You need to create an account here. click on click here. I have account that's why i added my details there.
once you create an account you will see this page.
Let's create an API key, By clicking on your profile and clicking settings.
Now click on API from the left side panel.
Now click on create
Click on Developer
Now you have to accept the terms and conditions.
Provide basic details
Click on submit and you will get your API key.

If TMDB is not working then
check other alternative of TMDB API Key and use it

## Step 4:
Install Prometheus
sudo useradd --system --no-create-home --shell /bin/false prometheus

wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
sudo mkdir -p /data /etc/prometheus
cd prometheus-2.47.1.linux-amd64/
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo chown -R prometheus:prometheus /etc/prometheus /data

sudo vim /etc/systemd/system/prometheus.service

[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle
Restart=on-failure

[Install]
WantedBy=multi-user.target

sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus


Access Prometheus:
Open http://<server-ip>:9090 in your browser.

## Install Node Exporter
sudo useradd --system --no-create-home --shell /bin/false node_exporter

wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/

sudo vim /etc/systemd/system/node_exporter.service

[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter
Restart=on-failure

[Install]
WantedBy=multi-user.target



sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter

sudo vim /etc/prometheus/prometheus.yml

- job_name: node_exporter
  static_configs:
    - targets: ["localhost:9100"]

curl -X POST http://localhost:9090/-/reload


## Install Grafana
sudo apt-get install -y apt-transport-https software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
sudo apt-get update

sudo apt-get install -y grafana
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server

Access Grafana:
Open http://<server-ip>:3000 in your browser.
Default credentials:
Username: admin
Password: admin


## Add Prometheus as a data source:                       

Go to Settings > Data Sources > Add data source.
Select Prometheus and enter http://localhost:9090.
Import a Dashboard:

Go to Dashboards > Import.
Use Dashboard ID 1860.
Select the Prometheus data source and import it.


## Step 5:
A} Install Prometheus Plugin in Jenkins

Ensure Jenkins is running.
Access Jenkins at http://<jenkins-ip>:8080.
Install the Prometheus Plugin:

Go to Manage Jenkins > Plugins > Available Plugins.
Search for Prometheus and install it.
Configure Prometheus in Jenkins:

After installation, Prometheus metrics will be available at /prometheus.
Go to Manage Jenkins > System Configuration > Prometheus, confirm the default settings, and click Apply and Save.


B} Add Jenkins as a Target in Prometheus
sudo vim /etc/prometheus/prometheus.yml

- job_name: 'jenkins'
  metrics_path: '/prometheus'
  static_configs:
    - targets: ['<jenkins-ip>:8080']

promtool check config /etc/prometheus/prometheus.yml

curl -X POST http://localhost:9090/-/reload

Check the targets:

Open http://<prometheus-server-ip>:9090/targets


C} Visualize Jenkins Metrics in Grafana
Import Jenkins Dashboard:

In Grafana, click Dashboards > + > Import.
Use Dashboard ID: 9964, then click Load.
Select the Prometheus data source:

Choose the Prometheus data source configured earlier.
View the Jenkins Dashboard:

After importing, you'll see a detailed overview of Jenkins metrics, such as:
Job success/failure rates
Build durations
Executor availability
System resource usage

Step 6:
A} 
Install Email Extension Plugin
Log in to Jenkins.
Go to Manage Jenkins → Manage Plugins.
Under the Available tab, search for Email Extension Plugin.
Select it, click Install without restart, and wait for the installation to complete.

B}
Generate an App Password in Gmail
Since Gmail requires secure access for third-party apps, you need to generate an App Password.

Log in to your Gmail account.
Click on your profile picture → Manage Your Google Account.
Go to the Security tab (from the left panel).
Ensure 2-Step Verification is enabled.
If not, set it up first.
Scroll down to App Passwords and click it.
Under Select app, choose Other (Custom name).
Enter a name like "Jenkins Email Integration" and click Generate.
A password will appear (16 characters, no spaces). Copy it and keep it safe.

C}
Configure Jenkins for Email Notifications
Go to Email Notification Settings:

In Jenkins, go to Manage Jenkins → Configure System.
Scroll to the E-mail Notification section.
Set the SMTP Server Details:

SMTP Server: smtp.gmail.com
Use SMTP Authentication: ✅ (Checked)
User Name: Your Gmail address (e.g., youremail@gmail.com)
Password: Paste the App Password generated earlier.
Check Use SSL.
SMTP Port: 465.
Click Apply and Save.

D}
Add Credentials for Gmail in Jenkins
Go to Manage Jenkins → Manage Credentials.
Under the appropriate domain (e.g., Global), click Add Credentials.
Add:
Kind: Username with password.
Username: Your Gmail address.
Password: The App Password.
ID (Optional): You can provide an ID for reference (e.g., gmail_credentials).

E}
Configure the Email Extension Plugin
Go back to Manage Jenkins → Configure System.
Scroll to Extended E-mail Notification.
Fill in the details:
SMTP Server: smtp.gmail.com
Default User E-mail Suffix: @gmail.com
Sender Email Address: Your Gmail address.
Use SSL: ✅ Checked.
SMTP Port: 465.
Click Test Configuration by Sending Test E-mail.
Enter your email address to verify the setup.
Click Apply and Save.

F}
 Add Email Notification to Jenkins Pipeline
Open your pipeline job.

Add a post section to your pipeline script for sending emails. Example:
  *********************Run jenkinsfile***********************

Step 7:
A}
Install Plugins
Access Plugin Manager
Log in to Jenkins.
Navigate to Manage Jenkins → Manage Plugins.
Go to the Available Plugins tab.


Install Required Plugins
Search for each of the following plugins and install them one by one:
Eclipse Temurin Installer (to manage Java installations).
SonarQube Scanner (to integrate Jenkins with SonarQube for static code analysis).
NodeJS Plugin (to manage Node.js versions for your builds).
OWASP Dependency-Check Plugin (for security analysis of dependencies).
Select each plugin and click Install without restart.
Wait for all plugins to install successfully.

B}
Configure Java and Node.js in Global Tool Configuration


Configure JDK (Java)
Go to Manage Jenkins → Global Tool Configuration.
Scroll down to JDK.
Click Add JDK:
Name: JDK 17 (or any descriptive name).
Install automatically: ✅ (Checked).
Select Eclipse Temurin and choose version 17.
Click Apply and Save.


Configure Node.js
Still in Global Tool Configuration, scroll down to NodeJS.
Click Add NodeJS:
Name: Node.js 16 (or any descriptive name).
Check Install automatically.
Select version 16.x.
Optionally, enable Global npm packages to install and specify additional packages, e.g., npm@latest.
Click Apply and Save.


## Step 8:
A}Configure SonarQube Server in Jenkins

A.
 Grab Public IP and Token from SonarQube
Public IP Address: Get the public IP address of your EC2 instance where SonarQube is running. Since SonarQube uses port 9000, you will access it via http://<EC2-Public-IP>:9000.
Generate Token:
Log in to your SonarQube server.
Navigate to Administration → Security → Users.
Click on Tokens and click on Update Token.
Enter a name for your token (e.g., "Jenkins Integration Token") and click Generate Token.
Copy the Token for use in Jenkins.


B.
 Add Token in Jenkins
Go to the Jenkins Dashboard.
Navigate to Manage Jenkins → Credentials → (Global) → Add Secret Text.
Secret: Paste the token you copied from SonarQube.
ID: Provide a recognizable ID (e.g., Sonar-token).
Description: Enter a description (e.g., "SonarQube Authentication Token").
Click OK.


C.
Configure SonarQube in Jenkins
Navigate to Manage Jenkins → Configure System.
Scroll down to the SonarQube Servers section.
Click Add SonarQube.
Enter the details:
Name: sonar-server (or any other name you prefer).
Server URL: http://<EC2-Public-IP>:9000 (replace <EC2-Public-IP> with the actual public IP).
Authentication Token: Select the Sonar-token credential you added earlier.
Click Apply and Save.

D.
Add SonarQube Quality Gate
Go to SonarQube Dashboard → Administration → Configuration → Webhooks.
Click on Create.
In the URL section, enter
http://<Jenkins-Public-IP>:8080/sonarqube-webhook/

B}
Create the Pipeline Job in Jenkins


A.
 Create a New Pipeline Job
From Jenkins’ dashboard, click on New Item.
Enter a name for your job (e.g., Netflix-Pipeline).
Select Pipeline and click OK.


B.
 Configure the Pipeline Script
In the pipeline configuration page:
Under the Pipeline section, select Pipeline script.
Paste the following declarative pipeline script:

***** Run jenkinsfile01 *****

C}
Run the Pipeline

Go back to your Jenkins job page and click on Build Now.


The job will start running, performing the following:
Cleaning workspace.
Checking out code from the Git repository.
Running SonarQube analysis.
Checking the quality gate result from SonarQube.
Installing dependencies via npm.


Pipeline Stages Breakdown:
Clean Workspace: Cleans up any files from the previous builds.
Checkout from Git: Pulls the latest code from the Git repository.
SonarQube Analysis: Runs static code analysis using SonarQube.
Quality Gate: Waits for the SonarQube quality gate result. If the code doesn't meet the quality standards, the build may fail.
Install Dependencies: Installs the necessary npm dependencies.


D}
View Results
Once the build is complete, you can check:

The SonarQube server (at http://<SonarQube-IP>:9000) to view the analysis results and quality gate status.
On Jenkins, you can see the build logs and email notifications (if configured).
SonarQube Reports:

In SonarQube, go to Projects to view the reports generated during the analysis.
You can view the status of your project (e.g., passed/failed) and inspect issues in detail.


Step 9:
A}
Install the OWASP Dependency Check Plugin
Navigate to Plugins:
Go to the Jenkins Dashboard → Manage Jenkins → Manage Plugins.
Under the Available Plugins tab, search for OWASP Dependency-Check.
Select the plugin and click Install without restart.

B}
Configure the OWASP Dependency Check Tool
Set Up the Tool:
Navigate to Dashboard → Manage Jenkins → Global Tool Configuration.
Scroll to Dependency-Check.
Add a new installation:
Name: Provide a recognizable name (e.g., DP-Check).
Configure additional options as needed.
Click Apply and Save.

C: Add OWASP Dependency Check to Your Pipeline
To integrate OWASP Dependency Check into your Jenkins pipeline:

Modify the Pipeline Script:
Add the following stages to your pipeline script:



"""
stage('OWASP FS SCAN') {
    steps {
        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    }
}

stage('TRIVY FS SCAN') {
    steps {
        sh "trivy fs . > trivyfs.txt"  // Trivy File System Scan
    }
}

"""


D}
Build and Review Results
Trigger the Build:

Go to your Jenkins pipeline job.
Click Build Now to execute the pipeline.
View Results:

OWASP Dependency Check results will appear as an XML file (dependency-check-report.xml).
Use Jenkins or a compatible tool to view the detailed report.
The Trivy scan results will be saved in trivyfs.txt.
