pipeline {
    agent any
    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }
    stages {
        stage('Fetch Code') {
            steps {
                git branch: 'master', url: 'https://github.com/chanelskil/TwoTier-Flask-App-Deployment.git'
            }
        }
        stage('Build and Test') {
            steps {
                sh 'echo "Run build and tests"'
                // Here, you'd run actual build and test commands
            }
        }
        stage('Deploy - Ansible Playbook') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'clientapp-key-ec2', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        export PATH=$PATH:/usr/bin
                        ansible-playbook -i ansible/hosts ansible/deploy.yml
                    '''
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    // Perform a simple health check, adjust the command to your app's health check URL or method
                    def response = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://172.31.60.148:5000", returnStdout: true).trim()
                    if (response == '200') {
                        echo "Application is running. Ending build with success."
                        currentBuild.result = 'SUCCESS'
                        // Force exit the pipeline successfully
                        error('Build marked successful as application is up and running.')
                    } else {
                        echo "Application health check failed. Check deployment logs."
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'This will always run regardless of the build result.'
        }
        success {
            echo 'Build was successful!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
