pipeline {
    agent any 
    environment {
        EC2_IP = "65.1.212.94"
    }
    stages {
        stage('Git checkout code') {
            steps {
                git branch: 'main', 
                credentialsId: 'Github-credentials', 
                url: 'https://github.com/umapathy1729/backend-project.git'
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2_key2']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} '
                    set -e
                    rm -rf E-flow-Backend
                    git clone git@github.com:umapathy1729/backend-project.git

                    cd E-flow-Backend
                    git checkout Umapathy
                    git pull origin Umapathy

                    docker stop backend_container || true
                    docker rm -rf backend_container || true
                    docker rmi backend_image || true

                    docker build -t backend_image .
                    docker run -d -p 3000:3000 --name backend_container backend_image

                    '
                    '''
                }
            }
        }
    }
}
