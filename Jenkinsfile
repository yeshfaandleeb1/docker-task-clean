pipeline {
    agent any

    environment {
        SONAR_SCANNER_HOME = tool 'DefaultScanner'
    }

    stages {

        /* ================================
           CHECKOUT CODE
        ================================= */
        stage('Checkout') {
            steps {
                echo "Checking out code..."
                git url: 'https://github.com/yeshfaandleeb1/docker-task-clean.git', branch: 'main'
            }
        }

        /* ================================
           SONARQUBE SCAN
        ================================= */
        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('SonarQube-Local') {
                        sh """
                            ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                              -Dsonar.projectKey=docker-task \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=http://192.168.1.36:9000 \
                              -Dsonar.token=${SONAR_TOKEN}
                        """
                    }
                }
            }
        }

        /* ================================
           DISABLED QUALITY GATE
        ================================= */
        stage('Quality Gate') {
            steps {
                echo "Skipping Quality Gate (Not supported in SonarQube Community Edition)"
            }
        }

        /* ================================
           DOCKER BUILD BACKEND
        ================================= */
        stage('Docker Build Backend') {
            steps {
                sh '''
                    echo "Building Backend Image..."
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
                    echo "Building Frontend Image..."
                    docker build -t greenx-frontend:${BUILD_NUMBER} \
                    ./GreenX_DCS_Assesment_Tool-main/greenX-assessment-tool-frontend
                '''
            }
        }

        /* ================================
           TRIVY SCAN + PUSH (PARALLEL)
        ================================= */
        stage('Scan & Push Images (Parallel Stage)') {
            steps {
                script {
                    parallel(

                        /* ---------- TRIVY SCAN ---------- */
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

                        /* ---------- DOCKER PUSH ---------- */
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

                                    echo "===== Pushing Images ====="
                                    docker push $DOCKER_USER/greenx-backend:${BUILD_NUMBER}
                                    docker push $DOCKER_USER/greenx-frontend:${BUILD_NUMBER}

                                    echo "===== PUSH COMPLETE ====="
                                '''
                            }
                        }
                    )
                }
            }
        }

        /* ================================
           7. DEPLOY USING DOCKER COMPOSE
        ================================= */
        stage('Deploy Using Docker Compose') {
            steps {
                echo "Deploying latest images using Docker Compose..."

                sh '''
                    docker compose -f docker-compose.images.yml pull
                    docker compose -f docker-compose.images.yml up -d
                '''

                echo """
                Deployment Completed Successfully!
                Vote Page   → http://localhost:8089
                Result Page → http://localhost:8088
                """
            }
        }

        /* ================================
           GIT PUSH BACK (FINAL STAGE)
        ================================= */
        stage('Git Push Updates') {
            steps {
                sh '''
                    echo "Pushing updates back to repository..."

                    git config --global user.email "yeshfaandleeb05@gmail.com"
                    git config --global user.name "jenkins"

                    git add .
                    git commit -m "Auto: Jenkins pipeline updates"
                    git push https://github.com/yeshfaandleeb1/docker-task-clean.git main || true
                '''
            }
        }
    }
}
