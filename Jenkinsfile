def gv

pipeline {
    agent any
    tools {
        maven 'maven-3.9'
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>' //this gives me an array containing the version info
                    def version = matcher[0][1] //this selects the first index item in the array, which will give me another array with the versionn tag and its children elements
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER" //this appends the Jenkins build number as the incremental version number
                }   
            }
        }
    
        stage('build app') {
            steps {
                script {
                    echo 'building the application...'
                    sh 'mvn clean package'
                }
            }
        }

        stage ('build image') {
            steps {
                script {
                    echo 'building the docker image...'
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "docker build -t learnq/demo-app:${IMAGE_NAME} ." //when using a variable need to enclose the shell script in double quotes
                        sh 'echo $PASS | docker login -u $USER --password-stdin'
                        sh "docker push learnq/demo-app:${IMAGE_NAME}"
                    }    
                }
            }
        }

        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image...'
                }
            }
        }

        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'gitlab-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh 'git status'
                        sh 'git branch'
                        sh 'git config --list'
                        sh "git remote set-url origin https://${USER}:${PASS}@gitlab.com/learn.ubq-group/08-jenkins.git"
                        sh 'git add .'
                        sh 'git commit -m "CI: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
    }
}