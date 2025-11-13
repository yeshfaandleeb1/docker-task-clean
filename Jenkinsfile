pipeline {
    agent any

    environment {
        BACKEND_DIR = "GreenX_DCS_Assesment_Tool-main/GreenX_DCS_Assesment_Tool_Backend"
        FRONTEND_DIR = "GreenX_DCS_Assesment_Tool-main/greenX-assessment-tool-frontend"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/yeshfaandleeb1/docker-task-clean.git'
            }
        }

        stage('Clean Workspace') {
            steps {
                echo "🧹 Cleaning workspace..."
                cleanWs()
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                echo "📦 Building backend..."
                dir("${BACKEND_DIR}") {
                    sh 'docker build -t greenx-backend:latest .'
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                echo "🖥️ Building frontend..."
                dir("${FRONTEND_DIR}") {
                    sh 'docker build -t greenx-frontend:latest .'
                }
            }
        }

        stage('List Docker Images') {
            steps {
                sh 'docker images'
            }
        }
    }

    post {
        success {
            emailext(
                to: 'yeshfaandleeb05@gmail.com',
                subject: "✅ Jenkins Pipeline Success — docker-task-clean",
                body: "Your Jenkins pipeline completed successfully."
            )
        }

        failure {
            emailext(
                to: 'yeshfaandleeb05@gmail.com',
                subject: "❌ Jenkins Pipeline FAILED — docker-task-clean",
                body: "Your Jenkins pipeline FAILED. Please check logs."
            )
        }
    }
}
