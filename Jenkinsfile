pipeline {

    agent any

    environment {
        SONARQUBE_URL = 'http://18.207.238.222:9000'
        BACKEND_DIR = 'GreenX_DCS_Assessment_Tool-main/GreenX_DCS_Assessment_Tool_Backend'
        FRONTEND_DIR = 'GreenX_DCS_Assessment_Tool-main/greenX-assessment-tool-frontend'
    }

    stages {

        /* ------------------------------
           1. Checkout code
        --------------------------------*/
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        /* ------------------------------
           2. SonarQube Scan
        --------------------------------*/
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

        /* --------------------------------
           3. Wait for Quality Gate
        --------------------------------*/
        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        /* --------------------------------
           4. Docker Build Backend
        --------------------------------*/
        stage('Docker Build Backend') {
            steps {
                sh """
                    docker build -t greenx-backend ./${BACKEND_DIR}
                """
            }
        }

        /* --------------------------------
           5. Docker Build Frontend
        --------------------------------*/
        stage('Docker Build Frontend') {
            steps {
                sh """
                    docker build -t greenx-frontend ./${FRONTEND_DIR}
                """
            }
        }

        /* --------------------------------
           6. Docker Push
        --------------------------------*/
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

        /* --------------------------------
           7. List Docker Images
        --------------------------------*/
        stage('List Docker Images') {
            steps {
                sh 'docker images'
            }
        }
    }

    /* --------------------------------
       Post Action: Email Notification
    --------------------------------*/
    post {
        always {
            emailext(
                to: 'yeshfaandleeb05@gmail.com',
                subject: "Jenkins Pipeline Finished",
                body: "Pipeline execution completed."
            )
        }
    }
}
