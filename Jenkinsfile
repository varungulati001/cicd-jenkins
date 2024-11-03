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
	stage('Maven Versioning') {
            steps {
                script {
                    sh 'mvn versions:set -DnewVersion=1.0.${BUILD_NUMBER}'
                }
                
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
 	stage('Build Docker Image and Tag') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker_cred', toolName: 'docker') {
                    sh 'docker build -t varungulati01/boardshack:latest .'
                }
            }
        }
     }  
}
}
