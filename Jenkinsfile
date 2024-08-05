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
                echo "Running build and tests"
                // Add actual test and build commands here
            }
        }
        stage('Pre-Deployment Check') {
            steps {
                script {
                    env.DEPLOY_NEEDED = 'true'
                    def responseCode = sh script: "curl -s -o /dev/null -w '%{http_code}' http://172.31.60.148:5000/health", returnStdout: true
                    if (responseCode == '200') {
                        echo 'Health check passed. Checking for updates...'
                        def currentCommit = sh script: "git rev-parse HEAD", returnStdout: true
                        def lastDeployedCommit = readFile('last_commit.txt').trim()
                        if (currentCommit == lastDeployedCommit) {
                            env.DEPLOY_NEEDED = 'false'
                            currentBuild.result = 'SUCCESS'
                            echo 'No updates to deploy.'
                        }
                    } else {
                        echo 'Health check failed or application is not responding as expected.'
                    }
                }
            }
        }
        stage('Deploy - Ansible Playbook') {
            when {
                expression { env.DEPLOY_NEEDED == 'true' }
            }
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'clientapp-key-ec2', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        export PATH=$PATH:/usr/bin
                        ansible-playbook -i ansible/hosts ansible/deploy.yml
                    '''
                }
            }
        }
    }
    post {
        success {
            echo 'Build was successful!'
        }
        failure {
            echo 'Build failed!'
        }
        always {
            // Additional cleanup or notifications can be added here.
        }
    }
}
