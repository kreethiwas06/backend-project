pipeline {
    agent any
 
    tools {
        // This matches the exact name configured in your Manage Jenkins -> Tools
        sonarScanner 'sonar-scanner-tool'
    }

    environment {
        // Define your Harbor registry details
        HARBOR_REGISTRY = '10.143.235.214'
        HARBOR_PROJECT  = 'demo2'
        IMAGE_NAME      = 'backend-service'
        IMAGE_TAG       = "${BUILD_NUMBER}" // Uses Jenkins build number as the tag
        
        // Credentials IDs configured in Jenkins Credentials Manager
        HARBOR_CREDS_ID = 'harbor-creds' 
        SONAR_SERVER_ID = 'sonar-token1' // Name of SonarQube server in Jenkins Global Config
    }
 
    stages {
        stage('Checkout') {
            steps {
                echo "Pulling code from GitHub..."
                git branch: 'main', url: 'https://github.com/umapathy1729/backend-project.git'
            }
        }
 
        stage('SonarQube Analysis') {
            steps {
                // Wraps the analysis in the SonarQube environment
                withSonarQubeEnv("${env.SONAR_SERVER_ID}") {
                    sh 'sonar-scanner'
                }
            }
        }
 
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:${env.IMAGE_TAG} ."
                }
            }
        }
 
        stage('Push to Harbor') {
            steps {
                script {
                    // Securely logs into Harbor using Jenkins credentials helper
                    withCredentials([usernamePassword(credentialsId: "${env.HARBOR_CREDS_ID}", usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASS')]) {
                        echo "Logging into Harbor..."
                        sh "echo '${HARBOR_PASS}' | docker login ${env.HARBOR_REGISTRY} -u '${HARBOR_USER}' --password-stdin"
                        
                        echo "Pushing image to Harbor..."
                        sh "docker push ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                        
                        // Optional: Tag as latest and push
                        sh "docker tag ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:${env.IMAGE_TAG} ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:latest"
                        sh "docker push ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:latest"
                    }
                }
            }
        }
 
        stage('Deploy with Docker Compose') {
            steps {
                script {
                    echo "Deploying application..."
                    sh "docker compose down"
                    sh "docker compose up -d"
                }
            }
        }
    }
 
    post {
        always {
            echo "Cleaning up local Docker images to save space..."
            sh "docker rmi ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:${env.IMAGE_TAG} || true"
            sh "docker rmi ${env.HARBOR_REGISTRY}/${env.HARBOR_PROJECT}/${env.IMAGE_NAME}:latest || true"
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Please check the logs."
        }
    }
}
