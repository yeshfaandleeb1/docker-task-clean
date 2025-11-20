pipeline {
    agent any

    environment {
        SONAR_HOST = "http://3.239.85.144:9000"
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
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh """
                        /opt/sonar-scanner/bin/sonar-scanner \
                          -Dsonar.projectKey=GreenX \
                          -Dsonar.projectName=GreenX \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=${SONAR_HOST} \
                          -Dsonar.login=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        stage("Sonar Quality Gate") {
            steps {
                timeout(time: 60, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t greenx_backend ./GreenX_DCS_Assesment_Tool_Backend'
                sh 'docker build -t greenx_frontend ./greenX-assessment-tool-frontend'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_creds',
                                                 usernameVariable: 'USER',
                                                 passwordVariable: 'PASS')]) {
                    sh """
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push greenx_backend
                    docker push greenx_frontend
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
        failure {
            emailext (
                to: "yeshfaandleeb05@gmail.com",
                subject: "Jenkins Build Failed",
                body: "Build failed. Check Jenkins console."
            )
        }
    }
}
