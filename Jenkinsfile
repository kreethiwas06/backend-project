pipeline {
    agent any

    environment {
        HARBOR_REGISTRY = '10.143.235.214'
        HARBOR_PROJECT  = 'demo2'
        IMAGE_NAME      = 'backend-service'
        IMAGE_TAG       = "${BUILD_NUMBER}"

        HARBOR_CREDS_ID = 'harbor-creds'
        SONAR_SERVER_ID = 'sonar-token1'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Pulling code from GitHub..."
                git branch: 'main',
                url: 'https://github.com/umapathy1729/backend-project.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {

                    // Gets Sonar Scanner path from Jenkins tools
                    def scannerHome = tool 'sonar-scanner-tool'

                    withSonarQubeEnv("${env.SONAR_SERVER_ID}") {

                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=backend-project \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=target
                        """
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."

                    sh """
                    docker build -t ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:${env.IMAGE_TAG} .
                    """
                }
            }
        }

        stage('Push to Harbor') {
            steps {
                script {

                    withCredentials([
                        usernamePassword(
                            credentialsId: "${env.HARBOR_CREDS_ID}",
                            usernameVariable: 'HARBOR_USER',
                            passwordVariable: 'HARBOR_PASS'
                        )
                    ]) {

                        echo "Logging into Harbor..."

                        sh """
                        echo '${HARBOR_PASS}' | docker login ${env.HARBOR_REGISTRY} \
                        -u '${HARBOR_USER}' --password-stdin
                        """

                        echo "Pushing Docker image..."

                        sh """
                        docker push ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:${env.IMAGE_TAG}
                        """

                        echo "Tagging latest image..."

                        sh """
                        docker tag \
                        ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:${env.IMAGE_TAG} \
                        ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:latest
                        """

                        sh """
                        docker push ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                script {

                    echo "Deploying application..."

                    sh "docker compose down || true"
                    sh "docker compose up -d"
                }
            }
        }
    }

    post {

        always {

            echo "Cleaning local Docker images..."

            sh """
            docker rmi ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:${env.IMAGE_TAG} || true
            """

            sh """
            docker rmi ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:latest || true
            """
        }

        success {
            echo "Pipeline completed successfully!"
        }

        failure {
            echo "Pipeline failed. Check console logs."
        }
    }
} 
