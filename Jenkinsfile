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

        /* 1️⃣ CHECKOUT */
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github_creds',
                    url: 'https://github.com/yeshfaandleeb1/GreenX_DCS_Assesment_Tool.git'
            }
        }

        /* 2️⃣ SONAR SCAN */
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonar') {
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=GreenX \
                          -Dsonar.projectName=GreenX \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://3.239.85.144:9000 \
                          -Dsonar.login=${SONAR}
                    """
                }
            }
        }

        /* 3️⃣ QUALITY GATE */
        stage("Sonar Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        /* 4️⃣ DOCKER BUILD */
        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${BACKEND_IMAGE}:latest ./GreenX_DCS_Assesment_Tool_Backend
                    docker build -t ${FRONTEND_IMAGE}:latest ./greenx-assessment-tool-frontend
                """
            }
        }

        /* 5️⃣ DOCKER PUSH */
        stage('Docker Push') {
            steps {
                sh """
                    echo ${DOCKERHUB_PSW} | docker login -u ${DOCKERHUB_USR} --password-stdin
                    docker push ${BACKEND_IMAGE}:latest
                    docker push ${FRONTEND_IMAGE}:latest
                """
            }
        }

        /* 6️⃣ IMAGE LISTING */
        stage('List Docker Images') {
            steps {
                sh "docker images"
            }
        }
    }

    /* 7️⃣ NOTIFICATIONS */
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
