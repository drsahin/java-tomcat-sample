# java-tomcat-maven-example

This is an example ready-to-deploy java web application built for Tomcat using Maven and webapp-runner.

## Running Locally

(need maven and java installed)

```
mvn package
java -jar target/dependency/webapp-runner.jar target/*.war
```

The application will be available on `http://localhost:8080`.

## How This Was Built

1. Generate the project using a Maven archetype:

   ```
   mvn archetype:generate -DarchetypeArtifactId=maven-archetype-webapp
   ```

2. Add the webapp-runner plugin into the `pom.xml`:

   ```
   <build>
     <!-- ... -->
     <plugins>
       <!-- ... -->
       <plugin>
         <groupId>org.apache.maven.plugins</groupId>
         <artifactId>maven-dependency-plugin</artifactId>
         <version>2.3</version>
         <executions>
           <execution>
             <phase>package</phase>
             <goals><goal>copy</goal></goals>
             <configuration>
               <artifactItems>
                 <artifactItem>
                   <groupId>com.github.jsimone</groupId>
                   <artifactId>webapp-runner</artifactId>
                   <version>8.5.11.3</version>
                   <destFileName>webapp-runner.jar</destFileName>
                 </artifactItem>
               </artifactItems>
             </configuration>
           </execution>
         </executions>
       </plugin>
     </plugins>
   </build>
   ```
# Jenkins-04 : Install & Configure Tomcat on Amazon Linux 2 AWS EC2 Instances

Purpose of the this hands-on training is to install & configure Tomcat server for staging and prodcution environment.

## Learning Outcomes

At the end of the this hands-on training, students will be able to;

- install and configure Tomcat server.


## Outline

- Part 1 - Launch 2 ec-2 free tier instances and Connect with SSH

- Part 2 - Install Java JDK

- Part 3 - Install Tomcat

- Part 4 - Configure Tomcat

- Part 5 - Auto start of tomcat server at boot

## Part 1 - Launch 2 ec2 free tier instances and Connect with SSH

- The security group must allow  SSH (port 22) and TCP (8080)

- connect to ec2 free tier instances 
  
```bash
ssh -i .ssh/mykey.pem ec2-user@ec2-3-133-106-98.us-east-2.compute.amazonaws.com
```

## Part 2 - Install Java JDK

- Install Java

- For Centos & Fedora (Amazon ec-2 instance)
```bash
sudo yum install java-1.8.0-openjdk -y
```

## Part 3 - Install Tomcat


- For Centos & Fedora (Amazon ec-2 instance)
  
```bash
sudo yum install unzip wget -y
```

- Install Tomcat

- Got to https://tomcat.apache.org/download-80.cgi page

- Look at Binary Distributions and copy the link of the `zip`ed one.

```bash
...
Core:
zip (pgp, sha512) [select this for linux, thus copy the link]
tar.gz (pgp, sha512)
32-bit Windows zip (pgp, sha512)
64-bit Windows zip (pgp, sha512)
32-bit/64-bit Windows Service Installer (pgp, sha512)
...
```

-  Get the tomcat file
  
```bash
cd /tmp
wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.63/bin/apache-tomcat-9.0.63.zip
```

- Unzip tomcat file and move to `/opt`
  
```bash
unzip apache-tomcat-*.zip
sudo mv apache-tomcat-9.0.63 /opt/tomcat
```

## Part 4 - Configure tomcat

- Now Change Tomcat Server Port

- Go to /opt/tomcat/conf/server.xml file

- Search for `Connector` and verify/change the Port Value, save the file.

```bash
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
```

- Change Permission of Scripts in `/opt/tomcat/bin`

```bash
cd /opt/tomcat/bin
ls -la
sudo chmod +x *
```

- Set Credentials of Tomcat that Jenkins will use.

```bash
cd /opt/tomcat/conf
```
- Update `tomcat-users.xml` file.

- `manager-script` & `admin-gui` are needed for jenkins to access tomcat.

- Set roles as `manager-script` & `admin-gui` and set password to tomcat as follows:

```bash
  <role rolename="manager-script"/>
  <role rolename="admin-gui"/>
  <user username="tomcat" password="tomcat" roles="manager-script,admin-gui"/>
```

- Note : Don't forget to remove the xml comment bloks `<!--` and `-->`. Delete these enclosing lines.

- To configure Tomcat server we need to modify the content of the context.xml. Be careful there are two of this file. We have to modify both of them.

- Go to the `/opt/tomcat/webapps/host-manager/META-INF/` and edit file `context.xml`. Actually commenting out the tagged `CookieProcessor` and `Valve` parts.

```bash
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<Context antiResourceLocking="false" privileged="true" >
	<!--
  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
                   sameSiteCookies="strict" />
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
	-->
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
```

- Go to the `/opt/tomcat/webapps/manager/META-INF/` and edit file `context.xml`. Actually commenting out the tagged `CookieProcessor` and `Valve` parts.

```bash
<?xml version="1.0" encoding="UTF-8"?>
<!--
  Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<Context antiResourceLocking="false" privileged="true" >
<!--
  <CookieProcessor className="org.apache.tomcat.util.http.Rfc6265CookieProcessor"
	sameSiteCookies="strict" />
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
-->
  <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
</Context>
```


- Restart the tomcat server

```bash
sudo /opt/tomcat/bin/shutdown.sh
sudo /opt/tomcat/bin/startup.sh
```

## Part 5 - Auto start of Tomcat server at boot

- In able to auto start Tomcat server at boot, we have to make it a `service`. Service is process that starts with operating system and runs in the background without interacting with the user.

- Service files are located in /etc/systemd/system.

- Go to /etc/systemd/system folder.

```bash
cd /etc/systemd/system
```

- In able to declare a service "unit file" must be created. Create a `tomcat.service` file.

```bash
sudo vi tomcat.service
```

- Copy and paste this code in "tomcat.service" file.
```
[Unit]
Description=Apache Tomcat Web Application Container
After=syslog.target network.target

[Service]
Type=forking

Environment=JAVA_HOME=/usr/lib/jvm/jre
Environment=CATALINA_PID=/opt/tomcat/temp/tomcat.pid
Environment=CATALINA_HOME=/opt/tomcat
Environment=CATALINA_BASE=/opt/tomcat
Environment='CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC'
Environment='JAVA_OPTS=-Djava.awt.headless=true -Djava.security.egd=file:/dev/./urandom'

ExecStart=/opt/tomcat/bin/startup.sh
ExecStop=/bin/kill -15 $MAINPID

[Install]
WantedBy=multi-user.target
```

- Save and exit.

- Enable Tomcat server.

```bash
sudo systemctl enable tomcat
```

- Start Tomcat server.

```bash
sudo systemctl start tomcat
```

- Open your browser, get your Tomcat server ec2 instance Public IPv4 DNS and paste it at address bar with 8080. 
"http://[ec2-public-dns-name]:8080"

# Jenkins-05 : Deploying Application to Staging/Production Environment with Jenkins

Purpose of the this hands-on training is to learn how to deploy applications to Staging/Production Environment with Jenkins.

## Learning Outcomes

At the end of the this hands-on training, students will be able to;

- deploy an application to Staging/Production Environment with Jenkins

- automate a Maven project as Pipeline.


## Outline

- Part 1 - Building Web Application

- Part 2 - Deploy Application to Staging Environment

- Part 3 - Update the application and deploy to the staging environment

- Part 4 - Deploy application to production environment

- Part 5 - Automate Existing Maven Project as Pipeline with Jenkins

## Part 1 - Building Web Application

- Fork the `https://github.com/JBCodeWorld/java-tomcat-sample.git` repo.

- Select `New Item`

- Enter name as `build-web-application`

- Select `Free Style Project`

- For Description : `This Job is packaging Java-Tomcat-Sample Project and creates a war file.`

- At `General Tab`, select Discard old builds, `Strategy` is `Log Rotation`, and for `Days to keep builds` enter `5` and `Max # of builds to keep` enter `3`.

- From `Source Code Management` part select `Git`

- Enter `https://github.com/<github-user-name>/java-tomcat-sample.git` for `Repository URL`.

- It is public repo, no need for `Credentials`.

- At `Build Environments` section, select `Delete workspace before build starts` and `Add timestamps to the Console Output` options.

- For `Build`, select `Invoke top-level Maven targets`

  - For `Maven Version`, select the pre-defined maven, `maven-3.8.4` 
  - For `Goals`, write `clean package`
  - POM: `pom.xml`

- At `Post-build Actions` section,
  - Select `Archive the artifacts`
  - For `Files to archive`, write `**/*.war` 

- Finally `Save` the job.

- Click `Build Now` option.

- Observe the Console Output

## Part 2 - Deploy Application to Staging Environment

- Select `New Item`

- Enter name as `Deploy-Application-Staging-Environment`

- Select `Free Style Project`

- For Description : `This Job will deploy a Java-Tomcat-Sample to the staging environment.`

- At `General Tab`, select Discard old builds, `Strategy` is `Log Rotation`, and for `Days to keep builds` enter `5` and `Max # of builds to keep` enter `3`.

- At `Build Environments` section, select `Delete workspace before build starts` and `Add timestamps to the Console Output` options.

- For `Build`, select `Copy artifact from another project`

  - Select `Project name` as `build-web-application`
  - Select `Latest successful build` for `Which build`
  - Check `Stable build only`
  - For `Artifact to copy`, fill in `**/*.war`

- For `Add post-build action`, select `Deploy war/ear to a container`
  - for `WAR/EAR files`, fill in `**/*.war`.
  - for `Context path`, filll in `/`.
  - for `Containers`, select `Tomcat 9.x Remote`.
  - Add credentials
    - Add -> Jenkins
      - Add `username` and `password` as `tomcat/tomcat`.
    - From `Credentials`, select `tomcat/*****`.
  - for `Tomcat URL`, select `private ip` of staging tomcat server like `http://172.31.20.75:8080`.

- Click on `Save`.

- Go to the `Deploy-Application-Staging-Environment` 

- Click `Build Now`.

- Explain the built results.

- Open the staging server url with port # `8080` and check the results.

## Part 3 - Update the application and deploy to the staging environment

-  Go to the `build-web-application`
   -  Select `Configure`
   -  Select the `Post-build Actions` tab
   -  From `Add post-build action`, `Build othe projects`
      -  For `Projects to build`, fill in `Deploy-Application-Staging-Environment`
      -  And select `Trigger only if build is stable` option.
   - Go to the `Build Triggers` tab
     - Select `Poll SCM`
       - In `Schedule`, fill in `* * * * *` (5 stars)
         - You will see the warning `Do you really mean "every minute" when you say "* * * * *"? Perhaps you meant "H * * * *" to poll once per hour`
  
   - `Save` the modified job.

   - At `Project build-web-application`  page, you will see `Downstream Projects` : `Deploy-Application-Staging-Environment`


- Update the web site content, and commit to the GitHub.

- Go to the  `Project build-web-application` and `Deploy-Application-Staging-Environment` pages and observe the auto build & deploy process.

- Explain the built results.

- Open the staging server url with port # `8080` and check the results.

## Part 4 - Deploy application to production environment

- Go to the dashboard

- Select `New Item`

- Enter name as `Deploy-Application-Production-Environment`

- Select `Free Style Project`

- For Description : `This Job will deploy a Java-Tomcat-Sample to the deployment environment.`

- At `General Tab`, select `Strategy` and for `Days to keep builds` enter `5` and `Max # of builds to keep` enter `3`.

- At `Build Environments` section, select `Delete workspace before build starts` and `Add timestamps to the Console Output` and `Color ANSI Console Outputoptions`.

- For `Build`, select `Copy artifact from another project`

  - Select `Project name` as `build-web-application`
  - Select `Latest successful build` for `Which build`
  - Check `Stable build only`
  - For `Artifact to copy`, fill in `**/*.war`

- For `Add post-build action`, select `Deploy war/ear to a container`
  - for `WAR/EAR files`, fill in `**/*.war`.
  - for `Context path`, filll in `/`.
  - for `Containers`, select `Tomcat 9.x Remote`.
  - From `Credentials`, select `tomcat/*****`.
  - for `Tomcat URL`, select `private ip` of production tomcat server like `http://172.31.28.5:8080`.

- Click on `Save`.

- Click `Build Now`.

## Part 5 - Automate Existing Maven Project as Pipeline with Jenkins

- Go to the Jenkins dashboard and click on `New Item` to create a pipeline.

- Enter `build-web-application-code-pipeline` then select `Pipeline` and click `OK`.

- Enter `This code pipeline Job is to package the maven project` in the description field.

- At `General Tab`, select `Discard old build`,
  -  select `Strategy` and 
     -  for `Days to keep builds` enter `5` and 
     -  `Max # of builds to keep` enter `3`.

- At `Advanced Project Options: Pipeline` section

  - for definition, select `Pipeline script from SCM`
  - for SCM, select `Git`
    - for `Repository URL`, select `https://github.com/<github-user-name>-tomcat-sample.git`, show the `Jenkinsfile` here.
    - for `Branch Specifier`, enter `*/main` as the GitHub branch is like that.
    - approve that the `Script Path` is `Jenkinsfile`
- `Save` and `Build Now` and observe the behavior.

- Copy the existing 2 jobs ( `Deploy-Application-Staging-Environment` , `Deploy-Application-Production-Environment` ) and modify them for pipeline.

- Go to dashbord click on `New Item` to copy `Deploy-Application-Staging-Environment`

- For name, enter `deploy-application-staging-environment-pipeline`

- At the bottom, `Copy from`, enter `Deploy-Application-Staging-Environment`

- Click `OK`, and `Save`

- Go to dashbord click on `New Item` to copy `Deploy-Application-Production-Environment`

- For name, enter `deploy-application-production-environment-pipeline`

- At the bottom, `Copy from`, enter `Deploy-Application-Production-Environment`

- Click `OK`, and `Save`


- Go to the `deploy-application-staging-environment-pipeline` job

- Find the `Build` section,
  - for `Project name`, enter `build-web-application-code-pipeline` 
  - select `Latest successful build`

- `Save` the job

- Go to the `deploy-application-production-environment-pipeline` job

- Find the `Build` section,
  - for `Project name`, enter `build-web-application-code-pipeline` 
  - select `Latest successful build`

- `Save` the job

- Now, go to the `build-web-application-code-pipeline` job and update the `Jenkinsfile` to include last 2 stages. For this purpose, add these 2 stages in `Jenkinsfile` like below:

```text
        stage('Deploy to Staging Environment'){
            steps{
                build job: 'deploy-application-staging-environment-pipeline'

            }
            
        }
        stage('Deploy to Production Environment'){
            steps{
                timeout(time:5, unit:'DAYS'){
                    input message:'Approve PRODUCTION Deployment?'
                }
                build job: 'deploy-application-production-environment-pipeline'
            }
        }
```

- Note: You can also use updated `Jenkinsfile2` file instead of updating `Jenkinsfile`.

- Go to the `build-web-application-code-pipeline` then select `Build Now` and observe the behaviors.
