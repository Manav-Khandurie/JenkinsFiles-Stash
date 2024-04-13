pipeline {
	agent any
	tools {
		maven "MAVEN3"
		jdk "OracleJDK11"
	}

	stages {
		stage('Fetch') {
			steps {
				git branch: 'main', url: 'https://github.com/hkhcoder/vprofile-project.git'
			}
		}

		stage('Build') {
			steps {
				sh 'mvn install -DskipTests'
			}

			post {
				success {
					echo 'Archieve Now...'
					archiveArtifacts artifacts : '**/*.war'
				}
			}
		}

		stage('Unit Tests') {
			steps {
				sh 'mvn test'
			}
		}
	}
}