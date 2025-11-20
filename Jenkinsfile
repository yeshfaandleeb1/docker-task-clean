pipeline {
    agent any

    environment {
        GITHUB = credentials('github_creds')
        DOCKERHUB = credentials('dockerhub_creds')
        SONAR = credentials('sonar_token')

        BACKEND_IMAGE = "yeshfaandleeb01/greenx-backend"
        FRONTEND_IMAGE = "yeshfaandleeb01/greenx-frontend"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github_creds',
                    url: 'https://github.com/yeshfaandleeb01/GreenX_DCS_Assesment_Tool.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=GreenX \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://localhost:9000 \
                          -Dsonar.login=${SONAR}
                    """
                }
            }
        }

        stage("Sonar Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${BACKEND_IMAGE}:latest ./GreenX_DCS_Assesment_Tool_Backend
                    docker build -t ${FRONTEND_IMAGE}:latest ./greenx-assessment-tool-frontend
                """
            }
        }

        stage('Docker Push') {
            steps {
                sh "echo ${DOCKERHUB_PSW} | docker login -u ${DOCKERHUB_USR} --password-stdin"
                sh "docker push ${BACKEND_IMAGE}:latest"
                sh "docker push ${FRONTEND_IMAGE}:latest"
            }
        }
    }

    post {
        success {
            emailext(
                subject: "SUCCESS: GreenX Jenkins Pipeline",
                body: "GreenX CI/CD Pipeline completed successfully.",
                to: "yeshfaandleeb05@gmail.com"
            )
        }
        failure {
            emailext(
                subject: "FAILED: GreenX Jenkins Pipeline",
                body: "Pipeline failed. Please check Jenkins logs.",
                to: "yeshfaandleeb05@gmail.com"
            )
        }
    }
}
