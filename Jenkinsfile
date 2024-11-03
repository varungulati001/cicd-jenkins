pipeline {
    agent any
	tools {
		maven 'maven3'
	}
	environment{
		SCANNER_HOME= tool 'sonar-scanner'
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

	stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                                                       -Dsonar.java.binaries=. -Dsonar.exclusions=**/trivy-fs-report.html '''
                }
            }
        }
	stage('Maven Build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Maven Deploy') {
            steps {
                sh 'mvn deploy'
            }
        }
}
}
