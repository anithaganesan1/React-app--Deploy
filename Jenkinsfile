pipeline {
    agent any

    tools {
        nodejs 'NodeJS_22' // Ensure this version exists in Jenkins tools
    }

    environment {
        IMAGE_NAME = "aniganesan/dev"
        DOCKER_HUB_CREDENTIALS = 'dockerhub-id'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo '📥 Checking out source code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('devops-build') {
                    echo '📦 Installing NPM packages...'
                    sh 'npm install'
                    sh 'chmod -R +x node_modules/.bin'
                }
            }
        }

        stage('Build React App') {
            steps {
                dir('devops-build') {
                    echo '🔨 Building React app...'
                    sh 'npm run build'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('devops-build') {
                    echo '🐳 Building Docker image...'
                    sh 'docker build -t $IMAGE_NAME .'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                dir('devops-build') {
                    echo '📤 Pushing image to Docker Hub...'
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push $IMAGE_NAME
                        """
                    }
                }
            }
        }
    }

    post {
        failure {
            echo '❌ Build or deployment failed!'
        }
        success {
            echo '✅ Build and deployment successful!'
        }
    }
}
