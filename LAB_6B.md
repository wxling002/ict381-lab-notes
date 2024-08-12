# Lab - SonarQube Integration with Jenkins

This lab will guide you through on integrating SonarQube with Jenkins. It covers the setup and configuration process, enabling you to leverage SonarQube's powerful code analysis capabilities within your Jenkins CI/CD pipeline. With SonarQube, you will be able to enhance the quality and maintainability of your codebase by automatically analyzing and tracking code issues, vulnerabilities and technical debt.

## Pre-requisites
1. A working Jenkins instance.

## Instructions
1. Run Sonarqube Docker container
2. Configure SonarQube
3. Create project token
4. Configure Jenkins
5. Configure StaycationX repo
6. Review Jenkins pipeline file
7. Create and configure webhook in Github
8. Create Jenkins Pipeline
9. Run Jenkins Pipeline and view Sonarqube Analysis

## Task 1: Run SonarQube Docker container

1. Login to the Jenkins EC2 instance.

2. Create a directory structure for SonarQube.

   ```bash
   mkdir -p /home/ubuntu/sonarqube
   cd /home/ubuntu/sonarqube
   mkdir data extensions logs pg_data pg_db
   ```
2. Run the following command to create a SonarQube docker container.

   ```bash
   cd /home/ubuntu/sonarqube
   git clone -b sonar git@github.com:USERNAME/staycationX
   cd staycationX
   docker-compose up -d
   ```

3. Navigate to the SonarQube website. Please allow some time for the page to be fully loaded.

   ```
   http://<JENKINS_EC2_IP>:9000
   ```

4. You should see the SonarQube login page.

   ![](images/lab6B/sonar-login.png)

5. Login with the default login username and password: `admin`.

6. You will be prompted to change the password. Enter a new password. Press **Update** to continue.

   |Field|Value|
   |---|---|
   |Old Password| admin|
   |New Password| Your preferred password|
   |Confirm Password| Re-enter your preferred password|

   ![](images/lab6B/sonar-update-password.png)

## Task 2: Configure SonarQube

1. On **How do you want to create your project?** page, click **Create a local project**.

2. On **Create local project** page, enter the following.

   |Field|Value|
   |---|---|
   |Project display name| staycationX|
   |Project key| staycationX |
   |Main branch name| main|

   ![](images/lab6B/sonar-create-local-project.png)

3. Click **Next** to continue.

4. On the **Set up project for Clean as You Code** page, click **Use the global setting** and **Create project**.

5. On **Analysis Method** page, click **With Jenkins**.

6. On **Select your DevOps platform**, click **Github**.

7. It will show the instructions on how to integrate SonarQube with Github. You may read the instructions shown.

## Task 3: Create project token

1. Click on **Account** logo on the top right corner.

2. Click **My Account**.

3. Click on **Security** tab.

4. Under **Generate Tokens** section, enter the following:

   |Field|Value|
   |---|---|
   |Name| staycationX-token|
   |Type| Select **Project Analysis Token** |

   ![](images/lab6B/sonar-create-token.png)

5. Click **Generate**.

6. The token will be displayed. Please be reminded to save the token for future use.

   ![](images/lab6B/sonar-token.png)


## Task 4: Configure Jenkins

#### Step 1: Install Plugin

1. Login to Jenkins.

2. From Dashboard, click **Manage Jenkins**.

3. Under **System Configuration**, click **Plugins**.

4. Click **Availabe Plugins** from the left menu, enter **Sonarqube Scanner** in the search box.

   ![](images/lab6B/sonar-scanner-search.png)

5. Check the checkbox and click **Install**.

6. Scroll down the page and ensure that you see **Success** for the plugin installation.

#### Step 2: Add SonarQube token to Jenkins

1. Click **Manage Jenkins** on the top menu breadcrumb.

2. Click **Credentials**.

3. Click **Global** link.

4. Click **+Add Credentials** button.

5. Under **Kind**, select **Secret text** from the dropdown list.

6. In the secret field, enter the value of the `staycationX-token` which was previously created.

7. In ID field, enter `sonarqube-token`.

8. Click **Create** button.

    ![](images/lab6B/jenkins-sonarqube-token.png)

9. You should see `sonarqube-token` added to Jenkins.

    ![](images/lab6B/jenkins-sonar-cred-added.png)


#### Step 3: Add Sonarqube server in Jenkins

1. Click **Manage Jenkins** on the top menu breadcrumb.

2. Under **System Configuration** section, click **System**.

3. Scroll down to **SonarQube servers** section, click **Add Sonarqube**.

4. Enter the following information.

   |Field|Value|
   |---|---|
   |Name| SonarServer|
   |Server URL| Enter the sonar IP address |
   |Server authentication token| Select `sonarqube-token` from the dropdown list|

   ![](images/lab6B/jenkins-sonarserver-add.png)

5. Click **Save**.

#### Step 4: Add SonarQube Scanner tool in Jenkins

1. Click **Manage Jenkins**.

2. Under **System Configuration** section, click **Tools**.

3. Under **SonarQube Scanner installations** section, click **Add SonarQube Scanner**

4. For **Name** field, enter **SonarScanner**.
   
   > **IMPORTANT**: The name should match the tool name in Jenkinsfile.

   ![](images/lab6B/jenkins-sonarqube-scanner-add.png)

5. Click **Save**.


## Task 5: Configure StaycationX repo

This task is for your information. The file has been populated in the repository.

1. In the StaycationX repo, switch to the `sonar` branch.

2. Add the file: `sonar-project.properties` under the root directory.

3. In the **sonar-project.properties** file, add the following content:

    ```bash
    sonar.projectKey=staycationX
    sonar.sources=.
    sonar.python.version=3.12
    ```

4. Save and push the changes into the `StaycationX` repo.


## Task 6: Review Jenkins pipeline file

1. Notice in the StaycationX repo, there is a `Jenkinsfile` in the root directory with the following content.

   This code snippet is a Jenkins pipeline stage for performing a SonarQube analysis.

   ```Jenkinsfile
     stage('SonarQube Analysis') {
          def scannerHome = tool 'SonarScanner';
          withSonarQubeEnv() {
          sh "${scannerHome}/bin/sonar-scanner"
          }
     }
   ```

## Task 7: Create and configure webhook in Github

When a new commit is pushed to the repository, the Jenkins pipeline will be triggered and the SonarQube analysis will be performed.

To enable this, you will need to create a webhook in the Github repository. 

If there is an existing webhook, you can skip this section.

1. Navigate to your Github `staycationX` repository.

2. Click on the **Settings** tab.

3. Under **Code and automation** section on the left, click **Webhooks**.

4. Click on **Add webhook**.

5. Enter the following details:
   - Payload URL: `http://<EC2_PUBLIC_IP>:8080/github-webhook/`
     > Replace the EC2_PUBLIC_IP with the jenkins machine public IP address
   - Content type: `application/json`
   - Which events would you like to trigger this webhook? Select `Just the push event`.

6. Click **Add webhook** to save the webhook.

7. You should see the webhook added to the repository.


## Task 8: Create Jenkins Pipeline

1. Click on the Jenkins logo on the top left to show the Dashboard.

2. Click **+ New Item** on the left menu shown in the Dashboard.

3. Under item name, give it a name: `sonar-pipeline`.

4. Select **Pipeline** and click **OK**.

5. Under **Build Triggers** section, select **Github hook trigger for GITScm polling**.

5. Under **Pipeline** section, choose **Pipeline script from SCM** from the Definition drop down list.

6. Select **Git** from the SCM dropdown list.

7. Under Repository URL, enter your Git repository in this format: `git@github.com:USERNAME/StaycationX`.

8. Under Branch Specifier, enter `*/sonar`.

9. Leave the rest as default and scroll down to the end of page and click **Save**.

## Task 9: Run Jenkins Pipeline and view Sonarqube Analysis

1. Click **Build Now**.

2. Under Build History, click on Build Run # link.

3. Click **Console Output** to view the build process.

4. Ensure the Build is successful.

   Showing part of the console output:

   ![](images/lab6B/sonar-console-output.png)


5. To view the results, head back to Sonarqube website. It should show you the analysis results.

If you are not able to get to this screen, click on the top left hand corner of the sonarqube logo, followed by clicking on `StaycationX` link.

   **Sample Results:**

   ![](images/lab6B/sonarqube-analysis.png)

   
   **Security Hotspots:**

   ![](images/lab6B/sonarqube-securityhotspots.png)


   **Issues:**

   ![](images/lab6B/sonarqube-issues.png)

---

**Congratulations!** You have completed the lab exercise.