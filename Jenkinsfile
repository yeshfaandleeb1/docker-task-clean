pipeline {
    agent any

    environment {
        GITHUB = credentials('github_creds')
        DOCKERHUB = credentials('dockerhub_creds')
        SONAR = credentials('sonar-token')

        BACKEND_IMAGE = "yeshfaandleeb01/greenx-backend"
        FRONTEND_IMAGE = "yeshfaandleeb01/greenx-frontend"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github_creds',
                    url: 'https://github.com/yeshfaandleeb1/docker-task-clean.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonar') {
                    sh """
                        /opt/sonar-scanner/bin/sonar-scanner \
                          -Dsonar.projectKey=GreenX \
                          -Dsonar.projectName=GreenX \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://3.239.85.144:9000 \
                          -Dsonar.login=${SONAR}
                    """
                }
            }
        }

        stage("Sonar Quality Gate") {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${BACKEND_IMAGE}:latest ./GreenX_DCS_Assement_Tool-main/GreenX_DCS_Assement_Tool_Backend
                    docker build -t ${FRONTEND_IMAGE}:latest ./GreenX_DCS_Assement_Tool-main/greenx-assessment-tool-frontend
                """
            }
        }

        stage('Docker Push') {
            steps {
                sh """
                    echo ${DOCKERHUB_PSW} | docker login -u ${DOCKERHUB_USR} --password-stdin
                    docker push ${BACKEND_IMAGE}:latest
                    docker push ${FRONTEND_IMAGE}:latest
                """
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
