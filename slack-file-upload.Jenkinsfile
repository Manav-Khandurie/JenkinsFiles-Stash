pipeline {
  agent any
  stages {
    stage('Slack') {
      steps {
        slackSend color: 'danger', message: 'test message'
      }
    }
    stage('create file') {
      steps {
        sh 'echo Hello World! > hello.txt'
      }
    }
    stage('Upload file') {
      steps {
        slackUploadFile filePath: 'hello.txt', initialComment: 'Here is your file'
      }
    }
  }
}