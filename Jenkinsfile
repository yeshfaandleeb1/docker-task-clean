pipeline {
    agent any

    environment {
        SONARQUBE_URL = "http://3.239.85.144:9000"
        PROJECT_KEY = "GreenX"
        PROJECT_NAME = "GreenX"
    }

    stages {

        /* -----------------------------------------------------------
           1. CHECKOUT
        ----------------------------------------------------------- */
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/yeshfaandleeb1/docker-task-clean.git',
                    credentialsId: 'github_creds'
            }
        }

        /* -----------------------------------------------------------
           2. SONARQUBE ANALYSIS
        ----------------------------------------------------------- */
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonar') {
                    withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_LOGIN')]) {

                        sh """
                            /opt/sonar-scanner/bin/sonar-scanner \
                                -Dsonar.projectKey=$PROJECT_KEY \
                                -Dsonar.projectName=$PROJECT_NAME \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=$SONARQUBE_URL \
                                -Dsonar.login=$SONAR_LOGIN
                        """
                    }
                }
            }
        }

        /* -----------------------------------------------------------
           3. SONAR QUALITY GATE CHECK
        ----------------------------------------------------------- */
        stage('Sonar Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        /* -----------------------------------------------------------
           4. DOCKER BUILD (Backend + Frontend)
        ----------------------------------------------------------- */
        stage('Docker Build') {
            steps {
                script {
                    sh 'docker build -t greenx-backend ./GreenX_DCS_Assesment_Tool_Backend'
                    sh 'docker build -t greenx-frontend ./greenX-assessment-tool-frontend'
                }
            }
        }

        /* -----------------------------------------------------------
           5. DOCKER PUSH
        ----------------------------------------------------------- */
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub_creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag greenx-backend $DOCKER_USER/greenx-backend:latest
                        docker tag greenx-frontend $DOCKER_USER/greenx-frontend:latest
                        docker push $DOCKER_USER/greenx-backend:latest
                        docker push $DOCKER_USER/greenx-frontend:latest
                    """
                }
            }
        }

        /* -----------------------------------------------------------
           6. LIST DOCKER IMAGES
        ----------------------------------------------------------- */
        stage('List Docker Images') {
            steps {
                sh 'docker images'
            }
        }
    }

    /* -----------------------------------------------------------
       7. EMAIL NOTIFICATION
    ----------------------------------------------------------- */
    post {
        success {
            emailext (
                subject: "Jenkins Build Success: GreenX CI/CD",
                body: "Build completed successfully!",
                to: "yeshfaandleeb05@gmail.com"
            )
        }
        failure {
            emailext (
                subject: "Jenkins Build Failed: GreenX CI/CD",
                body: "Build failed. Check Jenkins console logs.",
                to: "yeshfaandleeb05@gmail.com"
            )
        }
    }
}
