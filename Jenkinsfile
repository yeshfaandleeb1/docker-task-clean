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
                // use the exact name from Jenkins → Configure System → SonarQube Servers
                withSonarQubeEnv('MySonar') { 
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_LOGIN')]) {
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

        stage('Sonar Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build Backend') {
            steps {
                sh 'docker build -t greenx-backend ./GreenX_DCS_Assesment_Tool_Backend'
            }
        }

        stage('Docker Build Frontend') {
            steps {
                sh 'docker build -t greenx-frontend ./greenX-assessment-tool-frontend'
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

        stage('List Docker Images') {
            steps {
                sh 'docker images'
            }
        }
    }

    post {
        always {
            emailext(
                to: 'yeshfaandleeb05@gmail.com',
                subject: "Jenkins Pipeline Finished",
                body: "The pipeline has completed."
            )
        }
    }
}
