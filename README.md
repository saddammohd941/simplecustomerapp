# SimpleCustomerApp - A Java Web Application

This project is a Java web application for learning to build, deploy, and validate web apps using Maven, Jenkins, Nexus, SonarQube, and Tomcat. It features a customer form, navigation bar, learning links, and a Google Map.

## Prerequisites

- **Git**: For cloning the repository.
- **Java 17**: JDK 17 (e.g., OpenJDK or Red Hat).
- **Maven 3.9.6**: For building the project.
- **Jenkins Server**: Configured at `<JENKINS_SERVER>`.
- **Nexus Server**: Repository manager at `<NEXUS_SERVER>`.
- **SonarQube Server**: SonarCloud (`https://sonarcloud.io`) with SonarScanner installed.
- **Tomcat 9.0.105**: Deployed on `<TOMCAT_SERVER>:8080` with Manager UI access.
- **Credentials**:
  - GitHub: `saddammohd941_credentials` for repository access.
  - Nexus: `admin`/`admin@123` (configurable as needed).
  - Tomcat:
    - `admin`/`admin123`
    - `deployer`/`deployer123`
    - `tomcat`/`tomcat123`
  - SonarCloud: Token (`sonar` credential in Jenkins).
- **Network Access**: Ports 8080 (Tomcat), 8081 (Nexus), 2222 (SSH to Tomcat server).

## Setup Instructions

### 1. Clone the Repository
```bash
git clone https://github.com/saddammohd941/simplecustomerapp.git
cd simplecustomerapp
```

### 2. Configure Maven Settings
Update `~/.m2/settings.xml` with Nexus credentials:
```bash
sudo vi /home/jenkins/.m2/settings.xml
```
Add:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <servers>
        <server>
            <id>nexus-server</id>
            <username>admin</username>
            <password>admin@123</password>
        </server>
    </servers>
</settings>
```
Verify:
```bash
cat /home/jenkins/.m2/settings.xml
```

### 3. Configure Nexus Repository
1. Log into Nexus at `<NEXUS_SERVER>` with `admin`/`admin@123`.
2. Create a new Maven repository:
   - Name: `maven-snapshots`
   - Type: Hosted
   - Version Policy: **Snapshot**
   - Layout Policy: Strict
3. Update credentials if different:
   - Modify `admin`/`admin@123` in `settings.xml` and Jenkins credentials (`nexus-credentials`).

### 4. Configure SonarQube/SonarScanner
- Ensure SonarScanner is installed at `/opt/sonar-scanner/bin` on the Jenkins server.
- Verify SonarCloud access with the `sonar` credential in Jenkins (token for `saddammohd941` organization).

### 5. Use the below code in sonarqube analysis report or in Jenkinsfile.

```groovy
                        sonar-scanner -X \\
                        -Dsonar.projectKey=simplecustomerapp \\
                        -Dsonar.projectName=simplecustomerapp \\
                        -Dsonar.projectVersion=2.0 \\
                        -Dsonar.organization=saddammohd941 \\
                        -Dsonar.sources=src/main/java \\
                        -Dsonar.tests=src/test/java \\
                        -Dsonar.java.binaries=target/classes \\
                        -Dsonar.junit.reportsPath=target/surefire-reports \\
                        -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml \\
                        -Dsonar.login=\$SONAR_TOKEN \\
                        -Dsonar.host.url=https://sonarcloud.io
```

### 5. Verify Tomcat Configuration
- SSH into the Tomcat server:
  ```bash
  ssh -p 2222 root@<TOMCAT_SERVER>
  ```
- Check `/opt/tomcat/conf/tomcat-users.xml`:
  ```bash
  cat /opt/tomcat/conf/tomcat-users.xml
  ```
  Ensure:
```xml
<tomcat-users ...>
      <role rolename="manager-gui"/>
      <role rolename="admin-gui"/>
      <role rolename="host-manager-gui"/>
      <role rolename="manager-script"/>
      <role rolename="manager-jmx"/>
      <role rolename="manager-status"/>
      <user username="admin" password="admin123" roles="manager-gui,manager-script,manager-jmx,manager-status,admin-gui,host-manager-gui"/>
      <user username="hostadmin" password="hostadmin123" roles="admin-gui,host-manager-gui"/>
      <user username="deployer" password="deployer123" roles="manager-gui,manager-script"/>
      <user username="tomcat" password="tomcat123" roles="manager-gui"/>
</tomcat-users>
```
- Update if needed:
  ```bash
  sudo vi /opt/tomcat/conf/tomcat-users.xml
  systemctl restart tomcat
  ```
**Standard Tomcat Roles**

| Role Name             | Included? | Assigned To User(s)                          | What It Grants Access To                          |
|-----------------------|-----------|----------------------------------------------|---------------------------------------------------|
| manager-gui           | Yes       | admin, deployer, tomcat                      | Application Manager GUI (/manager/html)           |
| manager-script        | Yes       | admin, deployer                              | Manager HTTP API & scripts (for CI/CD)            |
| manager-jmx           | Yes       | admin                                        | JMX access via Manager                            |
| manager-status        | Yes       | admin                                        | Status pages (/manager/status)                    |
| admin-gui             | Yes       | admin, hostadmin                             | Admin interface (rarely used)                     |
| host-manager-gui      | Yes       | **admin**, **hostadmin**                     | **Host Manager GUI** (/host-manager/html)         |

### 6. Set Up Jenkins
- Log into Jenkins at `<JENKINS_SERVER>`.
- Configure credentials:
  - `saddammohd941_credentials`: GitHub access.
  - `nexus-credentials`: Nexus `admin`/`admin@123`.
  - `tomcat-credentials`: SSH key for `root@<TOMCAT_SERVER>`.
  - `sonar`: SonarCloud token.
- Verify tools:
  - Maven: `MAVEN_3.9.6`
  - JDK: `JDK17`

## How to Execute the Pipeline

### Option 1: Deploy Through Jenkins Pipeline
1. **Create a Pipeline Job**:
   - In Jenkins, create a new pipeline job named `SimpleCustomerApp`.
   - Select **Pipeline** and point to the `Jenkinsfile` in the repository or copy it:
     ```bash
     cat Jenkinsfile
     ```

2. **Pipeline Stages**:
   - **Clone Code**: Clones `https://github.com/saddammohd941/simplecustomerapp.git`.
   - **SonarCloud**: Runs code analysis with SonarScanner.
   - **mvn build**: Builds `SimpleCustomerApp-1.0.0-SNAPSHOT.war`.
   - **Publish to Nexus**: Deploys to `maven-snapshots`.
   - **Deploy to Tomcat**: Copies WAR to `<TOMCAT_SERVER>:/opt/tomcat/webapps`.

3. **Trigger Build**:
   - Click **Build Now** in Jenkins.
   - Monitor console output for errors.
   - Check deployment:
     ```bash
     ssh -p 2222 root@<TOMCAT_SERVER> "ls -l /opt/tomcat/webapps/SimpleCustomerApp-1.0.0-SNAPSHOT.war"
     ```

### Option 2: Deploy Through Maven Project
1. **Build the Project**:
   ```bash
   cd /home/jenkins/workspace/simplecustomerapp
   mvn -B -DskipTests clean package
   ```
   - Verify WAR:
     ```bash
     ls -l target/SimpleCustomerApp-1.0.0-SNAPSHOT.war
     ```

2. **Deploy to Nexus**:
   - Ensure `pom.xml` has the distribution management section:
     ```xml
     <distributionManagement>
         <snapshotRepository>
             <id>nexus-server</id>
             <url><NEXUS_SERVER>/repository/maven-snapshots/</url>
         </snapshotRepository>
     </distributionManagement>
     ```
   - Deploy:
     ```bash
     mvn deploy -DskipTests
     ```
   - Verify in Nexus:
     ```bash
     curl -u admin:admin@123 <NEXUS_SERVER>/repository/maven-snapshots/com/javatpoint/SimpleCustomerApp/1.0.0-SNAPSHOT/maven-metadata.xml
     ```

3. **Deploy to Tomcat**:
   - Copy WAR to Tomcat:
     ```bash
     ssh -p 2222 root@<TOMCAT_SERVER> "systemctl stop tomcat"
     ssh -p 2222 root@<TOMCAT_SERVER> "rm -rf /opt/tomcat/webapps/SimpleCustomerApp-1.0.0-SNAPSHOT*"
     scp -P 2222 target/SimpleCustomerApp-1.0.0-SNAPSHOT.war root@<TOMCAT_SERVER>:/opt/tomcat/webapps/
     ssh -p 2222 root@<TOMCAT_SERVER> "chown tomcat:tomcat /opt/tomcat/webapps/SimpleCustomerApp-1.0.0-SNAPSHOT.war"
     ssh -p 2222 root@<TOMCAT_SERVER> "systemctl start tomcat"
     ```
   - Check logs:
     ```bash
     ssh -p 2222 root@<TOMCAT_SERVER> "tail -n 50 /opt/tomcat/logs/catalina.out"
     ```

## Validating the Application

### 1. Check Nexus Artifacts
```bash
curl -u admin:admin@123 <NEXUS_SERVER>/repository/maven-snapshots/com/javatpoint/SimpleCustomerApp/1.0.0-SNAPSHOT/maven-metadata.xml
```
- Verify timestamped artifacts (e.g., `1.0.0-20250613.XXXXXX-1.war`).

### 2. Access Tomcat Manager UI
- Open `<TOMCAT_SERVER>:8080/manager/html`.
- Login with:
  - `admin`/`admin123`
  - `deployer`/`deployer123`
  - `tomcat`/`tomcat123`
- Check:
  - **Server Status**: Tomcat 9.0.105, JVM 17, Linux.
  - **Applications**: `/SimpleCustomerApp-1.0.0-SNAPSHOT` (Running).
- If missing, check logs:
  ```bash
  ssh -p 2222 root@<TOMCAT_SERVER> "tail -n 100 /opt/tomcat/logs/catalina.out"
  ssh -p 2222 root@<TOMCAT_SERVER> "tail -n 100 /opt/tomcat/logs/localhost.2025-06-13.log"
  ```

### 3. Access Application UI
- Open:
  ```
  <TOMCAT_SERVER>:8080/SimpleCustomerApp-1.0.0-SNAPSHOT/
  <TOMCAT_SERVER>:8080/SimpleCustomerApp-1.0.0-SNAPSHOT/Customer.jsp
  <TOMCAT_SERVER>:8080/SimpleCustomerApp-1.0.0-SNAPSHOT/index.html
  <TOMCAT_SERVER>:8080/SimpleCustomerApp-1.0.0-SNAPSHOT/Learning.html
  ```
- Expected:
  - **index.html**: Navigation bar (`Home`, `Apply Now`, `Learning Links`).
  - **Customer.jsp**: Form for customer details.
  - **Learning.html**: Links and Google Map.
  - **Welcome.jsp**: Displays form data after submission.
- Test form submission:
  - Fill out `Customer.jsp` and submit.
  - Verify redirect to `Welcome.jsp`.

## Resolution Steps

### If Pipeline Fails
- **SonarCloud Errors**:
  - Verify SonarScanner path: `/opt/sonar-scanner/bin`.
  - Check `sonar` credential in Jenkins.
- **Nexus Deploy Errors**:
  - Ensure `maven-snapshots` repository exists.
  - Update `settings.xml` with correct credentials.
  - Check:
    ```bash
    cat /home/jenkins/.m2/settings.xml
    ```
- **Tomcat Deploy Errors**:
  - Verify WAR:
    ```bash
    ssh -p 2222 root@<TOMCAT_SERVER> "ls -l /opt/tomcat/webapps/SimpleCustomerApp-1.0.0-SNAPSHOT.war"
    ```
  - Redeploy manually (see Maven deployment steps).

### If Manager UI Access Fails
- Check `/opt/tomcat/conf/tomcat-users.xml` for correct credentials.
- Allow all IPs in `/opt/tomcat/webapps/manager/META-INF/context.xml`:
  ```bash
  ssh -p 2222 root@<TOMCAT_SERVER> "sed -i 's/allow=\"[^\"]*\"/allow=\".*\"/g' /opt/tomcat/webapps/manager/META-INF/context.xml"
  systemctl restart tomcat
  ```

## Project Structure

```
simplecustomerapp/
├── src/main/webapp/
│   ├── Customer.jsp
│   ├── index.html
│   ├── Learning.html
│   ├── Welcome.jsp
│   ├── css/
│   │   ├── cust_css/main.css
│   │   ├── Header.css
│   ├── images/
│   ├── WEB-INF/
│   │   ├── web.xml
│   │   ├── classes/com/room/sample/servlet/InsertCustomerServlet.class
│   │   ├── classes/com/room/sample/view/Customer.class
│   │   ├── lib/javax.mail-api-1.6.2.jar
├── pom.xml
├── sonar-project.properties
├── Jenkinsfile
├── README.md
```

## Notes

- Update Nexus credentials in `settings.xml` and Jenkins if different from `admin`/`admin@123`.
- For issues, check Jenkins console or Tomcat logs.



### Next Steps
- **Update Repository**:
  - Save the `README.md` and `Jenkinsfile`:
    ```bash
    cd /home/jenkins/workspace/simplecustomerapp
    cat > README.md << 'EOF'
    [Paste the content from above, excluding <xaiArtifact> tags]
    EOF
    ```
  - Commit:
    ```bash
    git add README.md Jenkinsfile
    git commit -m "Update README.md without IPs or student references"
    git push origin main
    ```
- **Run Pipeline**:
  - Replace `<JENKINS_SERVER>`, `<NEXUS_SERVER>`, `<TOMCAT_SERVER>` in `Jenkinsfile` with actual IPs.
  - Trigger the pipeline to deploy `SimpleCustomerApp-1.0.0-SNAPSHOT`.
  - Verify at `<TOMCAT_SERVER>:8080/SimpleCustomerApp-1.0.0-SNAPSHOT/`.
- **Test Maven Deployment**:
  - Update `pom.xml` with the correct Nexus URL.
  - Follow Maven deployment steps.
  - Check Nexus and Tomcat.
- **Confirm Nexus Repository**:
  - Verify `maven-snapshots` exists in Nexus.
  - Check artifacts:
    ```bash
    curl -u admin:admin@123 <NEXUS_SERVER>/repository/maven-snapshots/com/javatpoint/SimpleCustomerApp/1.0.0-SNAPSHOT/maven-metadata.xml
    ```
