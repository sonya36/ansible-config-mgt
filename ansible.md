### Configuring Jenkins for Ansible Deployment on AWS

The following guide walks through setting up Jenkins on an AWS EC2 Ubuntu instance and configuring it to execute Ansible playbooks via a pipeline. By following these steps, you will learn how to set up a Jenkins server, connect it to GitHub, and create a Jenkins pipeline with a `Jenkinsfile` to streamline deployment tasks.

---

### Step 1: Launch an AWS EC2 Instance and Install Jenkins

1. **Launch an Ubuntu EC2 instance:**
   - Go to the AWS console, launch a new EC2 instance with Ubuntu as the operating system. Note down the public IP address or Elastic IP assigned to it.
   
2. **Update the instance and install Jenkins:**
   - SSH into the instance and run the following commands to install Jenkins and its dependencies.

   ```bash
   sudo apt-get update  # Update the instance
   ```

3. **Add Jenkins Repository Key:**
   Download the Jenkins repository key and add it to your keyrings.
   ```bash
   curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | gpg --dearmor | sudo tee /usr/share/keyrings/jenkins.gpg > /dev/null
   ```

4. **Add the Jenkins Repository:**
   Add the Jenkins repository to your sources list.
   ```bash
   echo "deb [signed-by=/usr/share/keyrings/jenkins.gpg] https://pkg.jenkins.io/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
   ```

5. **Update Package Index:**
   After adding the Jenkins repository, update the package list.
   ```bash
   sudo apt update
   ```

6. **Install Java (if not already installed):**
   Jenkins requires Java to run. Install OpenJDK 17:
   ```bash
   sudo add-apt-repository ppa:openjdk-r/ppa
   sudo apt install openjdk-17-jdk -y
   ```

6. **Install Jenkins:**
   Now, install Jenkins.
   ```bash
   sudo apt-get install jenkins -y
   ```

7. **Start Jenkins and Enable at Boot:**
   - Ensure Jenkins starts on boot and confirm it is running.

   ```bash
   sudo systemctl enable jenkins
   sudo systemctl start jenkins
   sudo systemctl status jenkins
   ```

### Step 1.2: Open TCP Port 8080

1. **Open Port 8080 in AWS Security Groups:**
   - Go to your AWS console, select the security group associated with your Jenkins EC2 instance, and allow inbound traffic on TCP port 8080.

### Step 2: Access Jenkins Dashboard

1. **Access Jenkins through the browser:**
   - Open a browser and navigate to `<Jenkins-server-Elastic-IP>:8080` to open Jenkins.

2. **Unlock Jenkins:**
   - To complete the initial setup, locate the initial password with the following command:

   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

3. **Complete Jenkins Setup:**
   - Follow the guided setup steps on the Jenkins UI to install recommended plugins and create an admin user.

### Step 3: Install the Blue Ocean Plugin for Pipeline Visualization

1. **Install Blue Ocean:**
   - In the Jenkins dashboard, go to **Manage Jenkins** > **Manage Plugins**, search for "Blue Ocean," and install it.
   - Once installed, access it through **Open Blue Ocean** in the Jenkins sidebar to simplify pipeline creation.

### Step 4: Connect Jenkins with GitHub

1. **Generate a GitHub Personal Access Token:**
   - Log in to GitHub and go to **Settings** > **Developer settings** > **Personal access tokens**.
   - Click on "Generate new token" and configure the token with the following settings:
     - **Name:** A descriptive name for the token.
     - **Expiration:** Select a suitable expiration period or "No expiration" if needed.
     - **Scopes:** Select `repo` for accessing repositories.
   - Generate and copy the token, as you will need it to connect Jenkins to GitHub.

2. **Add GitHub Token to Jenkins:**
   - In Blue Ocean, start creating a new pipeline by selecting your GitHub repository and pasting the personal access token when prompted.

### Step 5: Create the Pipeline in Jenkins

1. **Create the Pipeline:**
   - In the Blue Ocean interface, select **Create a new pipeline**. Choose the GitHub repository to set up the pipeline.

2. **Exit Blue Ocean:**
   - After creating the pipeline, click on **Administration** to exit Blue Ocean. We will manually configure the `Jenkinsfile`.

### Step 6: Configure the `Jenkinsfile`

1. **Create a `Jenkinsfile` in the Project Repository:**
   - Open your Ansible project in Visual Studio Code or your preferred editor.
   - Inside the project directory, create a new folder named `deploy`, then add a file named `Jenkinsfile`.

2. **Add Pipeline Code to `Jenkinsfile`:**
   - Copy the following code snippet into the `Jenkinsfile`. This initial setup defines a basic pipeline with a single `Build` stage that outputs "Building Stage."

   ```groovy
   pipeline {
       agent any

       stages {
           stage('Build') {
               steps {
                   script {
                       sh 'echo "Building Stage"'
                   }
               }
           }
       }
   }
   ```

3. **Configure Jenkins to Use the `Jenkinsfile`:**
   - In the Jenkins dashboard, navigate to the pipeline configuration for the Ansible project.
   - Scroll to the **Build Configuration** section and specify the path to the `Jenkinsfile` as `deploy/Jenkinsfile`.

4. **Scan Repository and Trigger Build:**
   - Go back to the pipeline view and click **Scan Repository Now** to verify the Jenkinsfile.
   - Select **Build Now** to start the pipeline. Jenkins will run the pipeline, and you can view the console output to confirm the pipeline executed successfully.

### Step 7: Verify the Pipeline Execution

1. **Check Console Output:**
   - Go to **Build History** in the pipeline view and open the build details to view the console output.
   - You should see the message "Building Stage" in the output, confirming that the pipeline executed as expected.

---

This setup enables you to run Ansible tasks directly from Jenkins and streamlines CI/CD by allowing Jenkins to handle deployments through GitHub repositories. Going forward, you can extend the `Jenkinsfile` to add more stages (like `Test` and `Deploy`) to create a full deployment pipeline.