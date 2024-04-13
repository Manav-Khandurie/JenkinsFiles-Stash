def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]

pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
        registryCredential = 'ecr:us-east-1:awsiamuser'
        appRegistry = "247477386084.dkr.ecr.us-east-1.amazonaws.com/manav-ecr-repo"
        vprofileRegistry = "https://247477386084.dkr.ecr.us-east-1.amazonaws.com/"
        cluster = "vprofile"
        service = "vprofileappsvc"
        slack_channel = "my-proj-cicd" 
        criticalityThreshold = 80
        trivyReportPath = "${env.WORKSPACE}/trivy_report.json"
    }

    stages {
        stage('Fetch code') {
            steps {
                git branch: 'docker', url: 'https://github.com/devopshydclub/vprofile-project.git'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Generate Surefire Reports') {
            steps {
                script {
                    sh 'mvn surefire-report:report'
                }
                junit 'target/surefire-reports/**/*.xml'
            }
        }

        stage('Build App Image') {
            steps {
                script {
                    dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
                }
            }
        }

        stage('Trivy Vulnerability Scan') {
            steps {
                script {
                    echo "${env.WORKSPACE}"
                    sh "trivy image ${appRegistry}:${BUILD_NUMBER} --severity CRITICAL --no-progress --format json > ${trivyReportPath}"
                }
            }
        }
        stage('Print Workspace Contents') {
            steps {
                sh "ls -al ${env.WORKSPACE}"
            }
        }

        stage('Trivy Report Upload to Slack') {
            steps {
                script {
                    slackSend channel: slack_channel, message: "Trivy scan report for ${appRegistry}:${BUILD_NUMBER}"
                    slackUploadFile channel: slack_channel, filePath: "${env.WORKSPACE}/trivy_report.json"                
                }
            }
        }


        stage('Upload App Image') {
            steps {
                script {
                    docker.withRegistry( vprofileRegistry, registryCredential ) {
                        dockerImage.push("$BUILD_NUMBER")
                        dockerImage.push('latest')
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: slack_channel, 
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}
