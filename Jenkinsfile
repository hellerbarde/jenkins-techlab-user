@Library('jenkins-techlab-libraries') _

pipeline {
    agent any // with hosted env use agent { label env.JOB_NAME.split('/')[0] }
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        timeout(time: 10, unit: 'MINUTES')
        timestamps()  // Timestamper Plugin
    }
//    environment{
//        M2_SETTINGS = credentials('m2_settings')
//        KNOWN_HOSTS = credentials('known_hosts')
//        ARTIFACTORY = credentials('jenkins-artifactory')
//        ARTIFACT = "${env.JOB_NAME.split('/')[0]}-hello"
//        REPO_URL = 'https://artifactory.puzzle.ch/artifactory/ext-release-local'
//    }
    tools {
        jdk 'jdk11'
        maven 'maven35'
    }
    stages {
        stage('Build') {
            steps {
                milestone(10)  // The first milestone step starts tracking concurrent build order
                sh 'mvn -B -V -U -e clean verify -DskipTests'
            }
        }
        stage('Test') {
            steps {
                // Only one build is allowed to use test resources, newest builds run first
                lock(resource: 'myResource', inversePrecedence: true) {  // Lockable Resources Plugin
                    sh 'mvn -B -V -U -e verify -Dsurefire.useFile=false -DargLine="-Djdk.net.URLClassPath.disableClassPathURLCheck=true"'
                    milestone(20)  // Abort all older builds that didn't get here
                }
            }
            post {
                always {
                    archiveArtifacts 'target/*.?ar'
                    junit 'target/**/*.xml'  // JUnit Plugin
                }
            }
        }
        stage('Deploy') {
            steps {
                input "Deploy?"
                milestone(30)  // Abort all older builds that didn't get here
                sh 'echo Call me maybe?'
                //sh "mvn -s '${M2_SETTINGS}' -B deploy:deploy-file -DrepositoryId='puzzle-releases' -Durl='${REPO_URL}' -DgroupId='com.puzzleitc.jenkins-techlab' -DartifactId='${ARTIFACT}' -Dversion='1.0' -Dpackaging='jar' -Dfile=`echo target/*.jar`"

                //sshagent(['testserver']) {
                //    sh "ls -l target"
                //    sh "ssh -o UserKnownHostsFile='${KNOWN_HOSTS}' -p 2222 richard@testserver.vcap.me 'curl -O -u \'${ARTIFACTORY}\' ${REPO_URL}/com/puzzleitc/jenkins-techlab/${ARTIFACT}/1.0/${ARTIFACT}-1.0.jar && ls -l'"
                //}
            }
        }
    }
    post {
        always {
            notifyPuzzleChat()
        }
    }
}
