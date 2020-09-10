pipeline {
    agent any // with hosted env use agent { label env.JOB_NAME.split('/')[0] }
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timeout(time: 10, unit: 'MINUTES')
        timestamps()  // Requires the "Timestamper Plugin"
    }
    environment{
        ARTIFACT = "${env.JOB_NAME.split('/')[0]}-hello"
    }
    tools {
        jdk 'jdk11'
        maven 'maven35'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B -V -U -e clean verify -Dsurefire.useFile=false  -DargLine="-Djdk.net.URLClassPath.disableClassPathURLCheck=true"'
                sshagent(['artifact-ssh']) {  // SSH Agent Plugin, artifact-ssh references the SSH credentials 
                    sh 'ssh-keyscan -p 2222 openssh-server > ~/.ssh/known_hosts'
                    sh 'ssh -p 2222 puzzler@openssh-server "whoami"'
                    sh 'ssh -p 2222 puzzler@openssh-server "mkdir -p ~/jenkins-techlab/${ARTIFACT}/1.0/"' 
                    sh "scp -P 2222 target/*.jar  puzzler@openssh-server:~/jenkins-techlab/${ARTIFACT}/1.0/"
                }
                archiveArtifacts 'target/*.?ar'
                junit 'target/**/*.xml'  // Requires JUnit plugin
            }
        }
    }
}