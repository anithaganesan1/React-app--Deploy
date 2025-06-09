pipeline {
    agent any

    environment {
        DEV_IMAGE_NAME = "aniganesan/dev"
        PROD_IMAGE_NAME = "aniganesan/prod"
        DOCKER_CREDENTIALS_ID = "dockerhub-id"
    }

    tools {
        nodejs "NodeJS_22"  // Make sure this is configured in Jenkins Global Tool Configuration
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "📥 Checking out source code..."
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('devops-build') {
                    echo "📦 Installing NPM packages..."
                   // script {
                       // if (fileExists('package-lock.json')) {
                           // sh 'npm ci'
                       // } else {
                           // sh 'npm install'
                      //  }
                    //}
                   echo "📦 Current directory:"
                   sh 'pwd'
                   echo "📋 Listing files:"
                   sh 'ls -la'
                   echo "📦 Installing NPM packages..."
                   sh 'npm install'
                   echo "📋 Listing node_modules directory:"
                   sh 'ls -la node_modules || echo "node_modules folder not found"'
                   sh 'ls -la node_modules/.bin || echo ".bin folder not found inside node_modules"'

                }
            }
        }

        stage('Build React App') {
            steps {
                dir('devops-build') {
                    echo "🔨 Building React app..."
                    sh 'npm run build'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def branch = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
                    def imageName = (branch == "main" || branch == "master") ? PROD_IMAGE_NAME : DEV_IMAGE_NAME
                    env.IMAGE_TAG = "${imageName}:latest"

                    echo "🐳 Building Docker image: ${env.IMAGE_TAG}"
                    dir('devops-build') {
                        sh "docker build -t ${env.IMAGE_TAG} ."
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "🔐 Logging in and pushing Docker image..."
                    withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                        sh "docker push ${env.IMAGE_TAG}"
                        sh "docker logout"
                    }
                }
            }
        }

        stage('Deploy (Optional)') {
            when {
                expression { return false }
            }
            steps {
                dir('devops-build') {
                    echo "🚀 Deploying Docker container..."
                    sh './deploy.sh'
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment successful!"
            cleanWs()
        }
        failure {
            echo "❌ Build or deployment failed!"
            cleanWs()
        }
    }
}
