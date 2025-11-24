pipeline {
    agent any

    environment {
        // Path where Jenkins installed the SonarQube Scanner tool (DefaultScanner)
        SCANNER_HOME = tool 'DefaultScanner'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Local') {
                    sh "${SCANNER_HOME}/bin/sonar-scanner"
                }
            }
        }

        stage('Docker Smoke Test') {
            steps {
                sh 'docker version'
                sh 'docker ps'
            }
        }

        // later we'll add backend/frontend docker build stages here
    }
}
