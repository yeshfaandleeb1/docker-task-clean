pipeline {
    agent any

    tools {
        // Name must match Manage Jenkins → Tools
        sonarQubeScanner 'DefaultScanner'
    }

    environment {
        // Use the scanner tool path
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

        // your existing backend/frontend Docker build stages can stay below
    }
}
