pipeline {
    agent any

    environment {
        BACKEND_DIR = "GreenX_DCS_Assesment_Tool-main/GreenX_DCS_Assesment_Tool_Backend"
        FRONTEND_DIR = "GreenX_DCS_Assesment_Tool-main/greenX-assessment-tool-frontend"
    }

    triggers {
        githubPush()   // auto-trigger on each GitHub push
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/yeshfaandleeb1/docker-task-clean.git',
                    credentialsId: 'github-credentials'
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                echo "📦 Building Backend Image..."
                dir("${BACKEND_DIR}") {
                    sh "docker build -t greenx-backend:latest ."
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                echo "🌐 Building Frontend Image..."
                dir("${FRONTEND_DIR}") {
                    sh "docker build -t greenx-frontend:latest ."
                }
            }
        }

        stage('List Docker Images') {
            steps {
                sh "docker images"
            }
        }
    }

    post {
        success {
            emailext to: 'yeshfaandleeb05@gmail.com',
                subject: "SUCCESS: Jenkins Pipeline Build Completed ✔",
                body: "Your pipeline has successfully completed!"
        }

        failure {
            emailext to: 'yeshfaandleeb05@gmail.com',
                subject: "FAILED ❌: Jenkins Pipeline Build",
                body: "Your pipeline failed. Please check Jenkins logs."
        }
    }
}
