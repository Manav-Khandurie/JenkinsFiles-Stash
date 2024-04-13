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
    }

    stages {
        stage('Fetch code'){
            steps {
                git branch: 'docker', url: 'https://github.com/devopshydclub/vprofile-project.git'
            }
        }

        stage('Test'){
            steps {
                sh 'mvn test'
            }
        }

        stage('Build App Image') {
            steps {
                script {
                    dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
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

        // stage('Deploy to ecs') {
        //     steps {
        //         withAWS(credentials: 'awsiamuser', region: 'us-east-1') {
        //             sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
        //         }
        //     }
        // }
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
