pipeline {
    agent any

    environment {
        // SonarQube Scanner installation (Manage Jenkins → Tools)
        SCANNER_HOME = tool 'DefaultScanner'
        // Sonar token from Jenkins credentials (ID must match your secret text ID)
        SONAR_TOKEN  = credentials('sonar-token')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                // Inject SONAR_HOST_URL etc. from server config "SonarQube-Local"
                withSonarQubeEnv('SonarQube-Local') {
                    sh """
                    ${SCANNER_HOME}/bin/sonar-scanner \
                      -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Docker Smoke Test') {
            steps {
                sh 'docker version'
                sh 'docker ps'
            }
        }

        // later we will add backend/frontend docker build stages
    }
}
