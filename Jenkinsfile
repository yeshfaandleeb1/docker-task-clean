pipeline {
    agent any

    stages {

        /* ================================
           DOCKER BUILD BACKEND
        ================================= */
        stage('Docker Build Backend') {
            steps {
                sh '''
                    echo "===== Building Backend Image ====="
                    docker build -t greenx-backend:${BUILD_NUMBER} \
                    ./GreenX_DCS_Assesment_Tool-main/GreenX_DCS_Assesment_Tool_Backend
                '''
            }
        }

        /* ================================
           DOCKER BUILD FRONTEND
        ================================= */
        stage('Docker Build Frontend') {
            steps {
                sh '''
                    echo "===== Building Frontend Image ====="
                    docker build -t greenx-frontend:${BUILD_NUMBER} \
                    ./GreenX_DCS_Assesment_Tool-main/greenx-assessment-tool-frontend
                '''
            }
        }

        /* ================================
           TRIVY SCAN + DOCKER PUSH (PARALLEL)
        ================================= */
        stage('Scan & Push Images (Parallel Stage)') {
            steps {
                script {
                    parallel(

                        /* ---- TRIVY IMAGE SCAN ---- */
                        "Trivy Image Scan": {
                            sh '''
                                echo "===== Creating Trivy Reports Folder ====="
                                mkdir -p trivy-reports

                                echo "===== Scanning Backend Image ====="
                                trivy image --format template --template @/opt/trivy-templates/html.tpl \
                                  -o trivy-reports/trivy-backend-report.html greenx-backend:${BUILD_NUMBER}

                                echo "===== Scanning Frontend Image ====="
                                trivy image --format template --template @/opt/trivy-templates/html.tpl \
                                  -o trivy-reports/trivy-frontend-report.html greenx-frontend:${BUILD_NUMBER}
                            '''

                            archiveArtifacts artifacts: 'trivy-reports/*.html', fingerprint: true
                        },

                        /* ---- DOCKER PUSH ---- */
                        "Docker Push": {
                            withCredentials([usernamePassword(
                                credentialsId: 'dockerhub-credentials',
                                usernameVariable: 'DOCKER_USER',
                                passwordVariable: 'DOCKER_PASS'
                            )]) {

                                sh '''
                                    echo "===== Logging into Docker Hub ====="
                                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                                    echo "===== Tagging Images ====="
                                    docker tag greenx-backend:${BUILD_NUMBER} $DOCKER_USER/greenx-backend:${BUILD_NUMBER}
                                    docker tag greenx-frontend:${BUILD_NUMBER} $DOCKER_USER/greenx-frontend:${BUILD_NUMBER}

                                    echo "===== Pushing Images to DockerHub ====="
                                    docker push $DOCKER_USER/greenx-backend:${BUILD_NUMBER}
                                    docker push $DOCKER_USER/greenx-frontend:${BUILD_NUMBER}

                                    echo "===== ALL IMAGES PUSHED SUCCESSFULLY ====="
                                '''
                            }
                        }
                    )
                }
            }
        }
    }
}
