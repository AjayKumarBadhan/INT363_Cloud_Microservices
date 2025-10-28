# Spring Boot Pet Clinic Deployment on EC2 (Amazon Linux)

This guide walks you through deploying the Spring Boot Pet Clinic application on an Amazon Linux EC2 instance.

---

## Prerequisites
- An active AWS account
- An EC2 instance running Amazon Linux 2 or Amazon Linux 2023
- SSH access to your EC2 instance

---

## Step-by-Step Deployment Guide

### 1. Update System Packages
```bash
sudo yum update -y
```
**Purpose:** Updates all installed packages to their latest versions, ensuring security patches and bug fixes are applied. The `-y` flag automatically confirms all prompts.

---

### 2. Install Java Development Kit (JDK)
```bash
sudo yum install -y java-25-amazon-corretto-devel
```
**Purpose:** Installs Amazon Corretto 25 (Amazon's distribution of OpenJDK), which is required to compile and run Spring Boot applications. The `devel` package includes both the JRE and development tools like `javac`.

---

### 3. Verify Java Runtime Installation
```bash
java -version
```
**Purpose:** Confirms that the Java Runtime Environment (JRE) is properly installed and displays the installed version number.

---

### 4. Verify Java Compiler Installation
```bash
javac -version
```
**Purpose:** Confirms that the Java Compiler is properly installed and displays its version. This is essential for compiling Java source code.

---

### 5. Install Git Version Control
```bash
sudo yum install git -y
```
**Purpose:** Installs Git, which is needed to clone the Spring Pet Clinic repository from GitHub.

---

### 6. Verify Git Installation
```bash
git --version
```
**Purpose:** Confirms Git is properly installed and displays the installed version.

---

### 7. Create Project Directory
```bash
mkdir springboot_project
```
**Purpose:** Creates a dedicated directory to organize your Spring Boot projects, following best practices for project structure.

---

### 8. Navigate to Project Directory
```bash
cd springboot_project
```
**Purpose:** Changes the current working directory to the newly created project folder where we'll clone the repository.

---

### 9. Clone the Spring Pet Clinic Repository
```bash
sudo git clone https://github.com/spring-projects/spring-petclinic.git
```
**Purpose:** Downloads the complete Spring Pet Clinic source code from the official GitHub repository to your EC2 instance.

---

### 10. Navigate to the Cloned Repository
```bash
cd spring-petclinic
```
**Purpose:** Enters the spring-petclinic directory that was just cloned from GitHub.

---

### 11. List Directory Contents
```bash
ls -la
```
**Purpose:** Displays all files and directories (including hidden files) with detailed information such as permissions, ownership, size, and modification dates. Useful for verifying the clone was successful.

---

### 12. Navigate to Full Project Path
```bash
cd /home/ec2-user/springboot_project/spring-petclinic
```
**Purpose:** Ensures you're in the correct absolute path of the project directory, avoiding any potential navigation issues.

---

### 13. Remove Existing Build Artifacts
```bash
sudo rm -rf target
```
**Purpose:** Deletes the `target` directory (if it exists) which contains previously compiled classes and build artifacts. This ensures a clean build from scratch.

---

### 14. Fix File Ownership Permissions
```bash
sudo chown -R ec2-user:ec2-user .
```
**Purpose:** Recursively changes the ownership of all files and directories to the `ec2-user` account. This is necessary because previous `sudo` commands may have created files owned by root, which could cause permission issues during the build.

---

### 15. Build the Application
```bash
./mvnw package -DskipTests -Dcheckstyle.skip=true -Dmaven.gitcommitid.skip=true
```
**Purpose:** Uses the Maven Wrapper to compile the application and package it into an executable JAR file. The flags skip tests, code style checks, and git commit ID generation to speed up the build process.

---

### 16. Run the Spring Boot Application
```bash
./mvnw spring-boot:run
```
**Purpose:** Starts the Spring Boot application using Maven. The application will run on port 8080 by default and remain active in the terminal session.

---

## Configure Security Group for Access

### Step 1: Navigate to Security Groups
1. Go to the **AWS Console**
2. Navigate to **EC2** → **Security Groups**

**Purpose:** Access the firewall rules that control inbound and outbound traffic to your EC2 instance.

---

### Step 2: Select Your Instance's Security Group
1. Select the security group associated with your EC2 instance

**Purpose:** Identifies which security group needs to be modified to allow traffic to your application.

---

### Step 3: Add Inbound Rule for Port 8080
1. Click on **Inbound rules** → **Edit inbound rules** → **Add rule**
2. Configure the new rule:
   - **Type:** Custom TCP
   - **Port:** 8080
   - **Source:** Your IP address (recommended) or `0.0.0.0/0` (for public access during testing)

**Purpose:** Opens port 8080 on your EC2 instance's firewall, allowing HTTP traffic to reach the Spring Boot application. Using your specific IP is more secure than allowing access from anywhere (`0.0.0.0/0`).

---

### Step 4: Access the Application
Open your web browser and navigate to:
```
http://your-ec2-public-ip:8080
```

**Purpose:** Accesses the Spring Pet Clinic web interface through your EC2 instance's public IP address on port 8080.

---

## Important Notes

- **Finding your EC2 Public IP:** Go to EC2 Dashboard → Instances → Select your instance → Copy the "Public IPv4 address"
- **Keep the terminal open:** The application runs in the foreground. Closing the terminal will stop the application.
- **For production deployment:** Consider running the application as a background service using systemd or screen/tmux
- **Security consideration:** Restrict port 8080 access to your IP address only, not `0.0.0.0/0`, for better security

---

## Troubleshooting

**If the application doesn't start:**
- Verify Java is installed correctly: `java -version`
- Check if port 8080 is already in use: `sudo netstat -tuln | grep 8080`
- Review application logs in the terminal output

**If you can't access the application:**
- Verify the security group rule is saved and active
- Confirm you're using the correct public IP address
- Check if your organization's network blocks port 8080

---

## Next Steps

Once the application is running, you can:
- Explore the Pet Clinic features
- Make code modifications in the `src` directory
- Rebuild using `./mvnw package`
- Learn about Spring Boot application structure and configuration
