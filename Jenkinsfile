pipeline { 
    agent any

    environment {
        SONAR_SCANNER_HOME = tool 'DefaultScanner'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out code..."
                git url: 'https://github.com/yeshfaandleeb1/docker-task-clean.git', branch: 'main'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SonarQube-Local') {
                        sh """
                            ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                              -Dsonar.projectKey=docker-task \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=http://192.168.1.17:9000 \
                              -Dsonar.token=${SONAR_TOKEN}
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

        // ---------------------------
        // 4. Docker Build (FINAL)
        // ---------------------------
        stage('Docker Build') {
            steps {
                sh '''
                    echo "Building Docker images using BuildKit"
                    export DOCKER_BUILDKIT=1

                    echo "Building backend image..."
                    docker build -t greenx-backend:${BUILD_NUMBER} ./GreenX_DCS_Assement_Tool-main/GreenX_DCS_Assement_Tool_Backend

                    echo "Building frontend image..."
                    docker build -t greenx-frontend:${BUILD_NUMBER} ./GreenX_DCS_Assement_Tool-main/greenX-assessment-tool-frontend
                '''
            }
        }

    }
}
