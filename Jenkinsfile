pipeline {
    agent any
	tools {
		maven 'maven3'
	}
    stages {
        stage('Git Checkout') {
            steps {
                git branch: "prod", credentialsId: 'git_cred', url: 'https://github.com/varungulati001/cicd-jenkins.git'
            }
        }
        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Maven Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Trivy Scan') {
            steps {
                sh 'trivy fs --format table --output trivy-fs-report.html .'
            }
        }
}
}
