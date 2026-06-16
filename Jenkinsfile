pipeline {
    agent any 
    environment {
        EC2_IP = "3.7.63.232"
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
                    rm -rf backend-project
                    git clone git@github.com:umapathy1729/backend-project.git

                    cd backend-project
                    git checkout main
                    git pull origin main

                    docker stop backend_container1 || true
                    docker rm -rf backend_container1 || true
                    docker rmi backend_image || true

                    docker build -t backend_image .
                    docker run -d -p 3009:5000 --name backend_container2 backend_image

                    '
                    '''
                }
            }
        }
    }
}
