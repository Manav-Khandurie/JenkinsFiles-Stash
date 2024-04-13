def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
    'REPORT': '#FFFF00'
]

pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
        registryCredential = 'ecr:us-east-1:awsiamuser'
        appRegistry = "247477386084.dkr.ecr.us-east-1.amazonaws.com/devopsproj-todo"
        Registry = "https://247477386084.dkr.ecr.us-east-1.amazonaws.com/"
        cluster = "vprofile"
        service = "vprofileappsvc"
        slack_channel = "my-proj-cicd" 
    }

    stages {
        stage('Fetch code') {
            steps {
                git branch: 'master', url: 'https://github.com/Manav-Khandurie/To-do-app-Maven.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
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
                    dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER")
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
