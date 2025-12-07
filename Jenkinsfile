pipeline {
    agent any

    tools {
        maven "MAVEN3" 
        jdk "OracleJDK21" 
    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "172.31.42.202:8081" 
        NEXUS_REPOSITORY = "vprofile-repo"
        NEXUS_CREDENTIAL_ID = "nesuxlogin" 
        
        SONAR_SERVER_NAME = "sonarserver"
    }

    stages {
        stage('Fetch Code') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv(SONAR_SERVER_NAME) {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload Artifact') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: NEXUS_VERSION,
                    protocol: NEXUS_PROTOCOL,
                    nexusUrl: NEXUS_URL,
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: NEXUS_REPOSITORY,
                    credentialsId: NEXUS_CREDENTIAL_ID,
                    artifacts: [
                        [artifactId: 'vprofile-v2', classifier: '', file: 'target/vprofile-v2.war', type: 'war']
                    ]
                )
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'SUCCESS: Artifact uploaded to Nexus.'
            slackSend (
                channel: '#jenkinscicd', 
                color: 'good', 
                message: "SUCCESS: Job '${env.JOB_NAME}' (Build #${env.BUILD_NUMBER}) finished successfully!", 
                tokenCredentialId: 'slacktoken' // Явно вказуємо ID токена
            )
        }
        failure {
            echo 'FAILURE: Pipeline failed.'
            slackSend (
                channel: '#jenkinscicd', 
                color: 'danger', 
                message: "FAILED: Job '${env.JOB_NAME}' (Build #${env.BUILD_NUMBER}) failed.", 
                tokenCredentialId: 'slacktoken'
            )
        }
    }
}
