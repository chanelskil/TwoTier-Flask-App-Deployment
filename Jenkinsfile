pipeline {
    agent any
    environment {
        ANSIBLE_HOST_KEY_CHECKING = 'False'
    }
    stages {
        stage('Setup') {
            steps {
                script {
                    // Ensure the last_commit.txt exists to compare commits.
                    if (!fileExists('last_commit.txt')) {
                        writeFile file: 'last_commit.txt', text: 'dummy-hash'
                    }
                }
            }
        }
        stage('Fetch Code') {
            steps {
                git branch: 'master', url: 'https://github.com/chanelskil/TwoTier-Flask-App-Deployment.git'
                script {
                    // Update last_commit.txt with the current commit hash.
                    def latestCommit = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
                    writeFile file: 'last_commit.txt', text: "${latestCommit}"
                }
            }
        }
        stage('Build and Test') {
            steps {
                echo "Running build and tests"
                // Add actual build and test commands here.
            }
        }
        stage('Pre-Deployment Check') {
            steps {
                script {
                    env.DEPLOY_NEEDED = 'true'
                    def responseCode = sh script: "curl -s -o /dev/null -w '%{http_code}' http://3.95.32.136:5000/", returnStdout: true
                    if (responseCode == '200') {
                        echo 'Health check passed. Checking for updates...'
                        def currentCommit = sh script: "git rev-parse HEAD", returnStdout: true
                        def lastDeployedCommit = readFile('last_commit.txt').trim()
                        if (currentCommit == lastDeployedCommit) {
                            env.DEPLOY_NEEDED = 'false'
                            echo 'No updates to deploy. Exiting build...'
                            currentBuild.result = 'SUCCESS'
                            // Use error to stop the pipeline gracefully.
                            error('No deployment needed. Latest code already deployed.')
                        }
                    } else {
                        echo 'Health check failed or application is not responding as expected.'
                        currentBuild.result = 'FAILURE'
                        error('Health check failed.')
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
        always {
            echo 'Cleaning up...'
            // Add cleanup steps if necessary.
        }
        success {
            echo 'Build was successful!'
        }
        failure {
            echo 'Build failed!'
        }
        aborted {
            echo 'Build was aborted!'
        }
    }
}
