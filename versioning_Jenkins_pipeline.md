## Demo Project - Dynamically Increment Application Version in Jenkins Pipeline

This demo shows the process of dynamically incrementing the application version in a Jenkins pipeline.


## Steps to Dynamically Increment Application Version in Jenkins Pipeline

**An application has a version and each build tool e.g. Maven and Gradle, or package manager tool e.g. npm and yarn, will keep that version and will maintain that version in its own main build file**

    **In Gradle the build file is build.gradle, in Maven it is pom.xml, and in npm or yarn it is package.json file**
    **Dependencies and plugins are listed in these build files, as well as the version of the application that is running**

**"-SNAPSHOT" is a suffix you can add to your application version in order to give some more information - it is normally used for development versions e.g. if you are just testing the version but it won't be released, you can add it to your version name**

**You can also suffix the version with "RELEASE" e.g. "2.3.5.RELEASE"**

**Versioning is a decision that the developer team makes**

**Whenever you are building a new application release you need to increment the version e.g. in pom.xml file for Maven**


1. **I set the application version to be incremented automatically - this should always be done. Especially when building a CI/CD Jenkins pipeline, build automation for your application releases, you want to be able to automatically increase that version inside your build. I setup Jenkins so that when I committed changes to Jenkins, the Jenkins build pipeline incremented that version and released a new application (all automatically)**

**All the build tools or package managers each have commands or plugins that are used to increment the version**

**I executed the following command in the java-maven-app project directory**

```bash
mvn build-helper:parse-version versions:set -DnewVersion=\${parsedVersion.majorVersion}.\${parsedVersion.minorVersion}.\${parsedVersion.nextIncrementalVersion} versions:commit
```

**The mvn build-helper plugin is a plugin we have available out of the box when you use Maven build tool. The "parse-version" plugin command goes and finds the pom.xml file and finds the version tag within it. When it finds the version tag it parses the version inside into 3 parts: major, minor, and increment; because we need to know what the current version is so we can increment it**

**The "versions:set" plugin command will set the version between the version tags in the pom.xml file**

**The version in the pom.xml file was increased from 1.1.0-SNAPSHOT to 1.1.1**

2. **To increment the minor version after a more advanced and larger application change, I executed the following**

```bash
mvn build-helper:parse-version versions:set \
-DnewVersion=\${parsedVersion.majorVersion}.\${parsedVersion.nextMinorVersion}.\${parsedVersion.incrementalVersion} versions:commit
```

**The version in the pom.xml file changed from 1.1.1 to 1.2.1**

**If I was using npm I would use a plugin or script that reads the version from package.json file, increments it and saves it back into package.json. Gradle and yarn will do something similar**

**This example was only executed locally. However, in reality this will be used for a CI/CD pipeline, so the command should be executed by Jenkins when the aplication build process is running**

3. **Next, I incremented the application version through Jenkins**

**First, I opened the Jenkinsfile in VSCode and made some changes**

**I would then need to increment the version before the maven build step in the Jenkinsfile (before the JAR file is built). To do this I added a new stage to the Jenkinsfile before the "build app" stage called "increment version"**

```bash
stage('increment version') {
    steps {
        script {
            echo 'incrementing app version...'
            sh 'mvn build-helper:parse-version versions:set \
                -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                versions:commit'
        }
    }
{
```

**I then had to enter a dynamic Image tag for the docker Image build command in the Jenkinsfile "build image" section. I did this by using the variable "$IMAGE_NAME" after the Docker Hub repo name. I did the same for the docker push command**

**I then added the following code to extract the version name from the pom.xml file**

```bash
def matcher = readFile('pom.xml') =~ '<version>(.+)</version>' //this gives me an array
def version = matcher[0][1]
env.IMAGE_NAME = "$version-$BUILD_NUMBER"
```

4. **I then had to update the JAR file name in the Dockerfile and change it from one that was hardcoded to one that was dynamically assigned (to represent the incremented version)**

```bash
COPY ./target/java-maven-app-*.jar /usr/app/
CMD java -jar java-maven-app-*.jar
```

**The star represents any version**

**I then had to add a command to the Jenkinsfile to clear the old JAR file before building the new one - so there is only one JAR file (important as we will match the jar file for assigning image name)**

**I updated the "mvn package" shell command in the Jenkinsfile to "mvn clean package"**

**This created a new JAR file. The Docker Image was then built using the JAR file and was pushed to the Docker Hub repo**

5. **I then committed these changes to the Git repository in the jenkins-jobs branch, then I built a pipeline for the jenkins-jobs branch - the jenkins-jobs build was successfully completed**

**I then added code to the Jenkinsfile to make Jenkins commit the new pom.xml with the updated Image version back to the Git repo. So, if other developers wanted to to commit something to that branch, they first needed to fetch the pom.xml changed file that Jenkins committed, and then continue working from there**

**I added this stage to the Jenkinsfile**

```bash
stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'gitlab-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh 'git config user.email "learn.ubq@gmail.com"'
                        sh 'git config user.name "learn.ubq"'
                        sh 'git status'
                        sh 'git branch'
                        sh 'git config --list'
                        sh "git remote set-url origin https://${USER}:${PASS}@gitlab.com/learn.ubq-group/08-jenkins.git"
                        sh 'git add.'
                        sh 'git commit -m "CI: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
```

**I then committed these changes to the Gitlab repo and run the jenkins-jobs pipeline build again - the build successfully completed**

**I could now see the new pom.xml file was committed to the Gitlab repo at the end**

**Now the gitlab credentials have been set I could now remove it for the next build**

6. **Lastly, I ignored Jenkins commit for Jenkins pipeline trigger - so there is no automatic triggerring of a new build each time a commit happens on the git repository - so there is no Jenkins/Git commit loop**

**I installed a plugin - "Ignore Committer Strategy" - this plugin gives me more configuration options for my multibranch pipeline - to ignore commits in our Gitlab repo made from Jenkins**

**I then went to my multibranch configuration settings in Jenkins UI - under "Branch Sources" I saw the "Build Strategies" setting - I added an "Ignore Committer Strategy" and entered the email of an author who makes commits (to be ignored - pipeline will not be started for this type of commit). I also selected "allow builds for any other authors" which are not in the list of emails I provided. I then saved it**

**Now after making a change to the Jenkinsfile and committing it, there was no Jenkins/Git commit loop**
