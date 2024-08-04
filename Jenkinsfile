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
                sshagent(['clientapp-key-ec2']) {
                    sh '''
                    ansible-playbook -i "ubuntu@172.31.60.148," /path/to/your/deploy_flask_app.yml
                    '''
                }
                ansiblePlaybook(credentialsId: 'client-key', playbook: 'deploy.yml', inventory: 'hosts.ini')
            }
        }
    }
}