# DevOps Project from Scratch | Simple DevOps Project for Beginners

## Project Overview
This project demonstrates setting up a **DevOps pipeline from scratch** using **Maven with Jenkins** and deploying it on **Apache Tomcat Server**. It includes installation, configuration, and deployment steps on **Amazon Linux 2 (t2.micro EC2 instance)**.

## Prerequisites
Before you begin, ensure you have:
- An **AWS account** to launch an EC2 instance
- Basic knowledge of Linux commands
- Security Group configured to allow **ports 8080 (Jenkins) and 22 (SSH)**

## Step 1: Launch and Configure Jenkins Server
### 1.1 Launch EC2 Instance
- **AMI:** Amazon Linux 2
- **Instance Type:** t2.micro (Free Tier)
- **Security Group:** Allow inbound **8080 (Jenkins), 22 (SSH)**
- **Storage:** Minimum 8GB

### 1.2 Connect to EC2 via SSH
```sh
ssh -i your-key.pem ec2-user@your-public-ip
```

### 1.3 Install and Configure Jenkins
```sh
sudo yum update -y
sudo wget -O /etc/yum.repos.d/jenkins.repo \ 
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
sudo yum upgrade -y
sudo amazon-linux-extras install epel -y
sudo amazon-linux-extras install java-openjdk11 -y
sudo yum install java-11-amazon-corretto -y

sudo yum install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins
systemctl status jenkins
```

### 1.4 Verify Java Installation
```sh
java -version
javac -version
```

## Step 2: Configure Hostname (Optional)
```sh
cd /etc
sudo vim hostname  # Edit hostname if required
sudo reboot
```

## Step 3: Access Jenkins UI
- Open a browser and enter:
  ```
  http://your-public-ip:8080
  ```

## Step 4: Install and Configure Maven
### 4.1 Download and Extract Maven
```sh
sudo su  & cd ~
cd /opt
wget https://dlcdn.apache.org/maven/maven-3/3.9.3/binaries/apache-maven-3.9.3-bin.tar.gz
tar -xzvf apache-maven-3.9.3-bin.tar.gz
ls
mv apache-maven-3.9.3 maven
ll
cd maven/
cd bin/
./mvn -v  
```
> If Maven is not accessible outside the `bin` folder, configure environment variables.

### 4.2 Set Up Maven Environment Variables
```sh
cd ~
pwd
ll -a  # Show hidden files
vim .bash_profile
```
- Add the following lines below the second `fi` statement:
```sh
M2_HOME=/opt/maven
M2=/opt/maven/bin
JAVA_HOME=/usr/lib/jvm/java-11-openjdk-11.0.19.0.7-1.amzn2.0.1.x86_64
PATH=$PATH:$HOME/bin:$JAVA_HOME:$M2_HOME:$M2
```
- Apply the changes:
```sh
echo $PATH
source .bash_profile
echo $PATH
```
- Verify Maven installation:
```sh
mvn -v
```
> **Note:** Set `MAVEN_HOME=/opt/maven` in Jenkins under Maven Installations.

## Step 5: Configure Maven in Jenkins
### 5.1 Install Maven Plugin
- Navigate to **Manage Jenkins → Manage Plugins**
- Install **Maven Integration Plugin** (without restart)

### 5.2 Configure Java and Maven in Jenkins
- **Manage Jenkins → Global Tool Configuration**
  - Add **JDK installation** → Java 11 → Set JAVA_HOME path
  - Add **Maven installation** → Name: Maven → Set MAVEN_HOME to `/opt/maven` (Untick "Install automatically")

### 5.3 Install Git on Jenkins Server
```sh
yum install git -y
```

## Step 6: Create and Build a Maven Project in Jenkins
### 6.1 Create a New Maven Project
- Go to **Jenkins → New Item → Maven Project**
- Set **Source Code Management**:
  - Git Repository URL
  - Branch: `*/main`
- Set **Build**:
  - Root POM: `pom.xml`
  - Goals: `clean install`
- Apply & Save
- Click **Build Now**

### 6.2 Verify Build Artifacts
- Navigate to **Job → Workspace → webapp → target**
- Check for the `war` file (build artifact)

## Useful Links
- [Jenkins Installation Guide](https://www.jenkins.io/doc/book/installing/linux/)
- [Jenkins Setup on AWS](https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/)
- [Maven Installation Guide](https://maven.apache.org/install.html)

---
This README provides a **basic setup guide**. You can expand it by adding **Maven build steps, Apache Tomcat configuration, and CI/CD pipeline integration.** 🚀



## Step 3: Install and Configure Apache Tomcat
### 3.1 Install Java (if not installed)
```sh
sudo amazon-linux-extras install java-openjdk11 -y
java -version
```

### 3.2 Download and Extract Tomcat
```sh
cd /opt
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.76/bin/apache-tomcat-9.0.76.tar.gz
tar -xvzf apache-tomcat-9.0.76.tar.gz
mv apache-tomcat-9.0.76 tomcat
```

### 3.3 Start Tomcat Server
```sh
cd /opt/tomcat/bin
./startup.sh
```
- Access Tomcat at: `http://your-public-ip:8080`

### 3.4 Configure Tomcat for Jenkins Deployment
- Modify `context.xml` files to allow deployments:
```sh
vim /opt/tomcat/webapps/manager/META-INF/context.xml
vim /opt/tomcat/webapps/host-manager/META-INF/context.xml
```
- Comment out the restriction lines using `<!--` and `-->`

- Modify `tomcat-users.xml` to add users:
```sh
vim /opt/tomcat/conf/tomcat-users.xml
```
- Add the following lines at the end:
```xml
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<role rolename="manager-jmx"/>
<role rolename="manager-status"/>
<user username="admin" password="admin" roles="manager-gui, manager-script, manager-jmx, manager-status"/>
<user username="deployer" password="deployer" roles="manager-script"/>
```
- Restart Tomcat:
```sh
cd /opt/tomcat/bin
./shutdown.sh
./startup.sh
```

## Step 4: Configure Jenkins for CI/CD Pipeline
### 4.1 Install Required Plugins
- Navigate to **Manage Jenkins → Manage Plugins**
- Install **Maven Integration Plugin** and **Deploy to Container Plugin** (without restart)

### 4.2 Configure Global Tools
- **Manage Jenkins → Global Tool Configuration**
  - Add **JDK installation** → Java 11
  - Add **Maven installation** → Name: Maven → Set MAVEN_HOME to `/opt/maven`
  - Add **Git installation** → Install Git:
```sh
yum install git -y
```

### 4.3 Add Tomcat Deployment Credentials
- **Manage Jenkins → Credentials → System → Global Credentials**
- Add **Username/Password Credential**:
  - Username: `deployer`
  - Password: `deployer`
  - ID: `tomcat-credentials`

### 4.4 Create and Configure a Jenkins Job
- Go to **Jenkins → New Item → Maven Project**
- **Source Code Management:**
  - Git Repository URL
  - Branch: `*/main`
- **Build Steps:**
  - Root POM: `pom.xml`
  - Goals: `clean install`
- **Post-build Actions:**
  - **Deploy WAR to Container**:
    - WAR/EAR Files: `**/*.war`
    - Container: **Tomcat 8.x Remote**
    - Tomcat URL: `http://your-public-ip:8080/`
    - Credentials: `tomcat-credentials`
- **Apply & Save**
- Click **Build Now**

### 4.5 Verify Deployment
- Access **Tomcat Manager App**: `http://your-public-ip:8080/manager/html`
- Check if the web application is deployed.
- Click on the deployed web application to verify it works.

## Step 5: Automate CI/CD with Poll SCM
- Navigate to **Jenkins → Job → Configure**
- Enable **Poll SCM** and set the schedule:
```sh
* * * * *
```
- This checks the Git repository every minute for changes and triggers a build automatically.

# **CI/CD pipeline** using **Jenkins, Maven, and Tomcat** on **AWS EC2**. The pipeline builds, packages, and deploys the application automatically. 🚀
