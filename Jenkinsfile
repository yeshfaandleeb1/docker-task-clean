pipeline {
    agent any

    stages {

        stage('Docker Build Backend') {
            steps {
                sh '''
                    echo "Building Backend Image..."
                    docker build -t greenx-backend ./GreenX_DCS_Assesment_Tool-main/GreenX_DCS_Assesment_Tool_Backend
                '''
            }
        }

        stage('Docker Build Frontend') {
            steps {
                sh '''
                    echo "Building Frontend Image..."
                    docker build -t greenx-frontend ./GreenX_DCS_Assesment_Tool-main/greenX-assessment-tool-frontend
                '''
            }
        }

    }
}
