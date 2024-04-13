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
        vprofileRegistry = "https://247477386084.dkr.ecr.us-east-1.amazonaws.com/"
        cluster = "todoapp-cluster"
        service = "todoapp-service"
        slack_channel = "my-proj-cicd" 
        trivyReportPath = "${env.WORKSPACE}/trivy_report.json"
    }
    
  stages {
    stage('Fetch code'){
      steps {
        git branch: 'master', url: 'https://github.com/Manav-Khandurie/To-do-app-Maven.git'
      }
    }


    stage('Unit Test and Integration Test'){
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

    stage('Trivy Vulnerability Scan') {
            steps {
                script {
                    echo "${env.WORKSPACE}"
                    sh "trivy image ${appRegistry}:${BUILD_NUMBER} --severity CRITICAL --no-progress --format json > ${trivyReportPath}"
                }
            }
    }

    stage('Upload to slack'){
        steps {
                script {
                    slackUploadFile channel: slack_channel, filePath: '${env.WORKSPACE}/trivy_report.json', initialComment: 'Trivy report'
                }
        }
    }
    
    stage('Upload App Image to AWS ECR') {
          steps{
            script {
              docker.withRegistry( vprofileRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
          }
     }

    stage('Deploy to ecs') {
          steps {
        withAWS(credentials: 'awsiamuser', region: 'us-east-1') {
          sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
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