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
# java-tomcat-sample

## Part 1 - Install Java, Maven and Git packages

- Connect to the Jenkins Server 
  
- Install Java
  
```bash
sudo yum update -y
sudo amazon-linux-extras install java-openjdk11 -y
sudo yum install java-devel 
```

- Install Maven
  
```bash
sudo su
cd /opt
rm -rf maven
wget https://dlcdn.apache.org/maven/maven-3/3.8.5/binaries/apache-maven-3.8.5-bin.tar.gz
tar -zxvf $(ls | grep apache-maven-*-bin.tar.gz)
rm -rf $(ls | grep apache-maven-*-bin.tar.gz)
sudo ln -s $(ls | grep apache-maven*) maven
echo 'export M2_HOME=/opt/maven' > /etc/profile.d/maven.sh
echo 'export PATH=${M2_HOME}/bin:${PATH}' >> /etc/profile.d/maven.sh
exit
source /etc/profile.d/maven.sh
```
- Install Git
  
```bash
sudo yum install git -y
```

## Part 2 - Maven Settings

- Open Jenkins GUI on web browser

- Setting System Maven Path for default usage
  
- Go to `Manage Jenkins`
  - Select `Configure System`
  - Find `Environment variables` part,
  - Click `Add`
    - for `Name`, enter `PATH+EXTRA` 
    - for `Value`, enter `/opt/maven/bin`
- Save

- Setting a specific Maven Release in Jenkins for usage
  
- Go to the `Global Tool Configuration`
- To the bottom, `Maven` section
  - Give a name such as `maven-3.8.5`
  - Select `install automatically`
  - `Install from Apache` version `3.8.5`
- Save

## Part 3 - Creating Package Application - Free Style Maven Job

- Select `New Item`

- Enter name as `Package-Application`

- Select `Free Style Project`

- For Description : `This Job is packaging Java-Tomcat-Sample Project and creates a war file.`

- At `General Tab`, select Discard old builds, `Strategy` is `Log Rotation`, and for `Days to keep builds` enter `5` and `Max # of builds to keep` enter `3`.

- From `Source Code Management` part select `Git`

- Enter `https://github.com/JBCodeWorld/java-tomcat-sample.git` for `Repository URL`.

- Go to the web browser and check the branch name of the git project `https://github.com/JBCodeWorld/java-tomcat-sample.git`. Most of the time, deafult branch is `master` but there may be some exceptions. Enter the branch name (`main`) to the `Branch Specifier (blank for 'any')`. 

- It is public repo, no need for `Credentials`.

- At `Build Environments` section, select `Delete workspace before build starts` and `Add timestamps to the Console Output` options.

- For `Build`, select `Invoke top-level Maven targets`

  - For `Maven Version`, select the pre-defined maven, `maven-3.8.5` 
  - For `Goals`, write `clean package`
  - POM: `pom.xml`

- At `Post-build Actions` section,
  - Select `Archive the artifacts`
  - For `Files to archive`, write `**/*.war` 

- Finally `Save` the job.

- Select `Package-Application`

- Click `Build Now` option.

- Observe the Console Output
