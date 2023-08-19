def COLOR_MAP = [
    'SUCCESS': 'good',  // good stands for the color green in Slack
    'FAILURE': 'danger' // danger stands for the color red in Slack
]

pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }
    options{
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '10', daysToKeepStr: '', numToKeepStr: '10')
    }
    environment{
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "172.31.89.121:8081"
        NEXUS_GROUP_ID = "vprofile-grp-repo"
        NEXUS_REPOSITORY = "vprofile-repo"
        NEXUS_CREDENTIALS_ID = "NexusLogin"
        ARTIFACT_VERSION = "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}"
    }
    stages {
        stage('Fetch code'){
            steps {
                git branch: 'vp-rem', url: 'https://github.com/owusukd/vprofile-project.git'
            }
            post {
                success {
                    echo 'Fetch Successful!'
                }
            }
        }
        stage('Build Artifact'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Archiving artifact'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        stage('Unit Test'){
            steps {
                sh 'mvn test'
            }
            post {
                success {
                    echo 'Unit test successful'
                }
            }
        }
        stage('Integration Test'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
            post {
                success {
                    echo 'Integration test successful'
                }
            }
        }
        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Checkstyle Report'
                }
            }
        }
        stage('Sonar Analysis'){
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
                withSonarQubeEnv('sonarQube'){
                    sh ''' ${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage('Quality Gate'){
            steps {
                timeout(time: 1, unit: 'HOURS'){
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Publish to Nexus Repository'){
            steps {
                nexusArtifactUploader(
                    nexusVersion: NEXUS_VERSION,
                    protocol: NEXUS_PROTOCOL,
                    nexusUrl: NEXUS_URL,
                    groupId: NEXUS_GROUP_ID,
                    version: ARTIFACT_VERSION,
                    repository: NEXUS_REPOSITORY,
                    credentialsId: NEXUS_CREDENTIALS_ID,
                    artifacts: [
                      [artifactId: 'vprofile',
                      classifier: '',
                      file: 'target/vprofile-v2.war',
                      type: 'war']   
                    ]
                )
            }
        }
    }
    post { 
        always {
            echo 'Slack Notifications'
            slackSend channel: '#jenkinscicd',
                      color: COLOR_MAP[currentBuild.currentResult], 
                      message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"           
        }
    }
}
