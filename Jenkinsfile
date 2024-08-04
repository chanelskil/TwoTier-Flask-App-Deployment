pipeline {
    agent any
    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }
    stages {
        stage('Fetch Code') {
            steps {
                git branch: 'master', url:'https://github.com/chanelskil/TwoTier-Flask-App-Deployment.git'
            }
        }
        stage('Build and Test') {
            steps {
                sh 'echo "Run build and tests"'
            }
        }
        stage('Deploy - Ansible Playbook') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'clientapp-key-ec2', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        ansible-playbook -i ansible/hosts ansible/deploy.yml
                    '''
                }
            }
        }
    }
}
