pipeline {
    agent any

    tools {
        sonarScanner 'DefaultScanner'
    }

    environment {
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out code..."
                git branch: 'main',
                    url: 'https://github.com/yeshfaandleeb1/docker-task-clean.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=docker-task \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://192.168.1.17:9000 \
                          -Dsonar.login=$SONAR_TOKEN
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
