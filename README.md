# Java Web Calculator

## Overview
This project is a Java-based web calculator application built using Maven. It integrates with SonarQube for static code analysis to ensure code quality, security, and maintainability.

This README covers:
- Setting up SonarQube 24.12.0.100206 (using embedded H2 DB, no PostgreSQL)
- Configuring your Java project for SonarQube analysis
- Running static code analysis and troubleshooting tips

---

## Prerequisites

| Component        | Requirement                            |
|------------------|-------------------------------------|
| OS               | Ubuntu 22.04 LTS / Amazon Linux 2   |
| RAM              | Minimum 4 GB (8 GB recommended)      |
| Java             | OpenJDK 17 (required by SonarQube)  |
| SonarQube Version| 24.12.0.100206                       |
| Build Tool       | Maven                               |
| User Privileges  | sudo privileges                     |

---

## 1. SonarQube Installation & Configuration

### Step 1: Install Java 17
```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version
````

### Step 2: Create SonarQube User

```bash
sudo useradd -m -d /opt/sonarqube -r -s /bin/bash sonar
```

### Step 3: Download & Setup SonarQube

```bash
cd /opt
sudo apt install wget unzip -y
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-24.12.0.100206.zip
sudo unzip sonarqube-24.12.0.100206.zip
sudo mv sonarqube-24.12.0.100206 sonarqube
sudo chown -R sonar:sonar /opt/sonarqube
```

### Step 4: Configure SonarQube

Edit the sonar properties:

```bash
sudo nano /opt/sonarqube/conf/sonar.properties
```

Uncomment or add these lines to use the embedded H2 DB and expose the web interface:

```properties
# Use embedded H2 DB (default)
# sonar.jdbc.username=
# sonar.jdbc.password=
# sonar.jdbc.url=

# Enable web server to listen on all interfaces
sonar.web.host=0.0.0.0
sonar.web.port=9000
```

Save and exit.

### Step 5: Setup SonarQube as a Systemd Service

Create service file:

```bash
sudo nano /etc/systemd/system/sonarqube.service
```

Paste the following:

```ini
[Unit]
Description=SonarQube service
After=network.target

[Service]
Type=forking
User=sonar
Group=sonar
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
Restart=always
LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

Enable and start SonarQube:

```bash
sudo systemctl daemon-reload
sudo systemctl enable sonarqube
sudo systemctl start sonarqube
sudo systemctl status sonarqube
```

### Step 6: Access SonarQube Dashboard

Open your browser and visit:

```
http://<server-ip>:9000
```

Default credentials:

* Username: `admin`
* Password: `admin` (you will be prompted to change this on first login)

---

## 2. Configure Java Maven Project for SonarQube

### Step 1: Install Sonar Scanner CLI

```bash
sudo apt install unzip -y
cd /opt
sudo wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-6.2.0.61181-linux.zip
sudo unzip sonar-scanner-6.2.0.61181-linux.zip
sudo mv sonar-scanner-6.2.0.61181-linux sonar-scanner
```

Add to PATH:

```bash
sudo nano /etc/profile.d/sonar-scanner.sh
```

Add this line:

```bash
export PATH=$PATH:/opt/sonar-scanner/bin
```

Activate:

```bash
source /etc/profile.d/sonar-scanner.sh
sonar-scanner -v
```

### Step 2: Generate SonarQube Token

* Login to SonarQube dashboard
* Go to **My Account > Security > Generate Tokens**
* Create token named `maven-analysis`
* Copy token securely

### Step 3: Add SonarQube Properties to Maven Project

Create `sonar-project.properties` in the project root:

```properties
sonar.projectKey=java-web-app
sonar.projectName=Java Web Calculator
sonar.projectVersion=1.0
sonar.sources=src
sonar.language=java
sonar.java.binaries=target/classes
sonar.host.url=http://<server-ip>:9000
sonar.login=<your-generated-token>
```

### Step 4: Run Maven with SonarQube Analysis

```bash
mvn clean verify sonar:sonar \
  -Dsonar.projectKey=java-web-app \
  -Dsonar.host.url=http://<server-ip>:9000 \
  -Dsonar.login=<your-generated-token>
```

---

## 3. Troubleshooting

| Issue                    | Cause                           | Fix                               |
| ------------------------ | ------------------------------- | --------------------------------- |
| SonarQube not starting   | Insufficient RAM                | Allocate ≥4GB RAM                 |
| "Java not found"         | Incorrect JAVA_HOME environment | Set JAVA_HOME correctly           |
| Sonar scanner fails      | Invalid token or wrong URL      | Verify token and SonarQube URL    |
| Port 9000 already in use | Another app using port 9000     | Change `sonar.web.port` in config |

---

## 4. Useful Commands & History Highlights

```bash
java --version
mvn --version
sudo apt install openjdk-17-jdk -y
sudo apt install maven -y
git clone https://github.com/akracad/JavaWebCalculator.git
cd JavaWebCalculator/
mvn clean package
mvn clean verify sonar:sonar -Dsonar.projectKey=java-web-app -Dsonar.host.url=http://<server-ip>:9000 -Dsonar.login=<token>
sudo systemctl status sonarqube
sudo ss -tulnp | grep 9000
```

---

## 5. Summary

This setup allows seamless static code analysis using SonarQube with embedded H2 database and Java 17. It integrates directly with your Maven-based Java project and provides code quality insights such as bugs, vulnerabilities, code smells, and more via a clean web dashboard.

---

## Screenshots

*Add your screenshots below or link them here for reference.*


Would you like me to help add screenshot placeholders or instructions on how to embed them?
```
