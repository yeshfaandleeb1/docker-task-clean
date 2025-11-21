pipeline {
    agent any

    environment {
        SONARQUBE_URL = 'http://18.207.238.222:9000'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonar') {
                    withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_LOGIN')]) {
                        sh """
                        /opt/sonar-scanner/bin/sonar-scanner \
                          -Dsonar.projectKey=GreenX \
                          -Dsonar.projectName=GreenX \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=$SONARQUBE_URL \
                          -Dsonar.login=$SONAR_LOGIN
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build Backend') {
            steps {
                sh 'docker build -t greenx-backend ./GreenX_DCS_Assesment_Tool-main/GreenX_DCS_Assesment_Tool_Backend'
            }
        }

        stage('Docker Build Frontend') {
            steps {
                sh 'docker build -t greenx-frontend ./GreenX_DCS_Assesment_Tool-main/greenX-assessment-tool-frontend'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub_creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker tag greenx-backend $DOCKER_USER/greenx-backend:latest
                        docker tag greenx-frontend $DOCKER_USER/greenx-frontend:latest
                        docker push $DOCKER_USER/greenx-backend:latest
                        docker push $DOCKER_USER/greenx-frontend:latest
                    """
                }
            }
        }

        stage('Approval') {
            steps {
                script {
                    input message: 'Deploy to production?'
                }
            }
        }

        stage('Deploy using Docker Compose') {
            steps {
                sh 'docker compose -f docker-compose.yml up -d'
            }
        }
    }

    post {
        always {
            emailext(
                to: 'yeshfaandleeb05@gmail.com',
                subject: "Jenkins Pipeline Finished",
                body: "Pipeline completed successfully."
            )
        }
    }
}
