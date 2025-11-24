pipeline {
    agent any

    stages {
        stage('Docker Build') {
            steps {
                sh '''
                    echo "Building Docker images using BuildKit"
                    export DOCKER_BUILDKIT=1

                    echo "Building backend image..."
                    docker build -t greenx-backend:${BUILD_NUMBER} ./GreenX_DCS_Assesment_Tool-main/GreenX_DCS_Assesment_Tool_Backend

                    echo "Building frontend image..."
                    docker build -t greenx-frontend:${BUILD_NUMBER} ./GreenX_DCS_Assesment_Tool-main/greenX-assessment-tool-frontend
                '''
            }
        }
    }
}
