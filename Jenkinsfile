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
           QUALITY GATE (DISABLED)
        ================================= */
        stage('Quality Gate') {
            steps {
                echo "Skipping Quality Gate (Not supported in Community Edition)"
            }
        }

        /* ================================
           MANUAL APPROVAL
        ================================= */
        stage('Approval Required') {
            steps {
                script {
                    emailext(
                        to: "yeshfaandleeb05@gmail.com",
                        subject: "APPROVAL REQUIRED: GreenX Pipeline Build #${BUILD_NUMBER}",
                        body: """\
Hello,

Your GreenX CI/CD pipeline is waiting for manual approval.

Click this link to approve deployment:
${env.BUILD_URL}

Thanks,
Jenkins CI/CD System
"""
                    )

                    timeout(time: 30, unit: 'MINUTES') {
                        input message: "Approve Deployment?", ok: "Deploy Now"
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
           DEPLOY USING DOCKER COMPOSE
        ================================= */
        stage('Deploy Using Docker Compose') {
            steps {
                script {

                    echo "===== Removing Old Containers ====="
                    sh '''
                        docker rm -f greenx-frontend || true
                        docker rm -f greenx-backend || true
                    '''

                    echo "===== Pulling Latest Images ====="
                    sh """
                        docker compose -f docker-compose.images.yml pull
                    """

                    echo "===== Deploying Containers ====="
                    sh """
                        docker compose -f docker-compose.images.yml up -d
                    """
                }
            }
        }
    }

    /* ================================
       POST-EXECUTION EMAILS
    ================================= */
    post {
        success {
            emailext(
                to: "yeshfaandleeb05@gmail.com",
                subject: "SUCCESS: GreenX Build #${BUILD_NUMBER}",
                body: """\
Build #${BUILD_NUMBER} deployed successfully!

✔ Backend:  http://192.168.1.36:8001
✔ Frontend: http://192.168.1.36:8002

SonarQube Dashboard:
http://192.168.1.36:9000/dashboard?id=docker-task

Trivy Scan Reports:
Backend:  ${env.BUILD_URL}artifact/trivy-reports/trivy-backend-report.html
Frontend: ${env.BUILD_URL}artifact/trivy-reports/trivy-frontend-report.html

Regards,
Jenkins CI/CD System
"""
            )
        }

        failure {
            emailext(
                to: "yeshfaandleeb05@gmail.com",
                subject: "FAILED: GreenX Build #${BUILD_NUMBER}",
                body: """\
Build #${BUILD_NUMBER} failed!

Check Jenkins logs here:
${env.BUILD_URL}

Regards,
Jenkins CI/CD System
"""
            )
        }
    }
}
