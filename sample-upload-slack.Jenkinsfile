pipeline {
    agent any
    
    environment {
        slack_channel = "my-proj-cicd"
        file_name = "example_file.txt"
    }
    
    stages {
        stage('Create File') {
            steps {
                script {
                    writeFile file: "${file_name}", text: "This is an example text file."
                }
            }
        }
        
        stage('Upload File to Slack') {
            steps {
                script {
                    slackUploadFile channel: "${slack_channel}", filePath: "${file_name}"
                }
            }
        }
    }
}
