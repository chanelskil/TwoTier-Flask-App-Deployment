pipeline {
    agent any
    stages {
        stage('SCM') {
            steps {
                git 'https://github.com/chanelskil/TwoTier-Flask-App-Deployment.git'
            }
        }
        stage('Build and Test') {
            steps {
                sh 'echo "Run build and tests"'
            }
        }
        stage('Deploy') {
            steps {
                ansiblePlaybook(credentialsId: 'AWS_SSH', playbook: 'deploy.yml', inventory: 'hosts.ini')
            }
        }
    }
}

