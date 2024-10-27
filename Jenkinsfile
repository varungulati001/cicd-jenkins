pipeline {
    agent any
    stages {
        stage('Git Checkout') {
            steps {
                git branch: "prod", credentialsId: 'git-cred', url: 'https://github.com/varungulati001/cicd-jenkins.git'
            }
        }
}
}
