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
           SONARQUBE ANALYSIS
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
           SONAR QUALITY GATE
        ================================= */
        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        /* ================================
           APPROVAL REQUIRED BEFORE DEPLOY
        ================================= */
        stage('Manual Approval for Deploy') {
            steps {
                script {
                    timeout(time: 15, unit: 'MINUTES') {
                        input message: "Approval Required: Deploy to Production?", ok: "Deploy Now"
                    }
                }
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
           TRIVY SCAN & PUSH (PARALLEL)
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
           DEPLOY USING DOCKER COMPOSE
        ================================= */
        stage('Deploy to Production') {
            steps {
                sh '''
                    echo "========================================"
                    echo "     DEPLOYING USING DOCKER COMPOSE     "
                    echo "========================================"

                    docker compose -f docker-compose.images.yml down || true
                    docker compose -f docker-compose.images.yml pull || true
                    docker compose -f docker-compose.images.yml up -d

                    echo "==== DEPLOY FINISHED ===="
                '''
            }
        }
    }

    /* ================================
       POST EXECUTION EMAILS
    ================================= */
    post {
        success {
            emailext(
                to: "yeshfaandleeb05@gmail.com",
                subject: "SUCCESS: Build #${BUILD_NUMBER} Deployed Successfully!",
                body: """
Hello Team,

The pipeline has successfully completed and the application is deployed.

BUILD NUMBER: ${BUILD_NUMBER}

SONARQUBE DASHBOARD:
http://192.168.1.36:9000/dashboard?id=docker-task

TRIVY REPORTS:
Backend: http://192.168.1.36:8080/job/GreenX-CICD-Pipeline/${BUILD_NUMBER}/artifact/trivy-reports/trivy-backend-report.html
Frontend: http://192.168.1.36:8080/job/GreenX-CICD-Pipeline/${BUILD_NUMBER}/artifact/trivy-reports/trivy-frontend-report.html

LIVE APPLICATION:
Vote App: http://localhost:8089
Result App: http://localhost:8088

Regards,
Jenkins CI/CD Pipeline
                """
            )
        }

        failure {
            emailext(
                to: "yeshfaandleeb05@gmail.com",
                subject: "FAILED: Build #${BUILD_NUMBER}",
                body: """
Build FAILED.

Check Jenkins logs and fix the issue.

Pipeline: GreenX-CICD-Pipeline  
Build Number: ${BUILD_NUMBER}
                """
            )
        }
    }
}
