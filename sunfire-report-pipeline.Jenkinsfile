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
        appRegistry = "247477386084.dkr.ecr.us-east-1.amazonaws.com/manav-ecr-repo"
        vprofileRegistry = "https://247477386084.dkr.ecr.us-east-1.amazonaws.com/"
        cluster = "vprofile"
        service = "vprofileappsvc"
        slack_channel = "my-proj-cicd" 
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

        stage('Upload Surefire Reports to Slack') {
            steps {
                slackUploadFile filePath: 'target/site/surefire-report.html', initialComment: 'Here is your file'
                // script {
                //     def reportPath = "${WORKSPACE}/target/site/surefire-report.html"
                //     def fileName = 'surefire-report.html'
                //     //echo "~~~ $reportpath"
                //     slackSend(
                //         channel: slack_channel,
                //         message: "Surefire Reports for ${env.JOB_NAME} build ${env.BUILD_NUMBER}",
                //         attachments: [
                //             [
                //                 title: fileName,
                //                 text: "Surefire Report",
                //                 color: COLOR_MAP['REPORT'],
                //                 file: reportPath, // Directly specify the file path
                //                 fileType: 'html'
                //             ]
                //         ],
                //         tokenCredentialId: "Slack_Token_jenkins-cicd-lab.slack.com"
                //     )
                // }
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
