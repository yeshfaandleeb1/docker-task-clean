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
           MANUAL APPROVAL BEFORE BUILD
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

Click this link to approve the deployment:
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
           SKIP QUALITY GATE (Community Edition)
        ================================= */
        stage('Quality Gate') {
            steps {
                echo "Skipping Quality Gate (Not supported in Community Edition)"
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
                                mkdir -p trivy-reports

                                trivy image --format template --template @/opt/trivy-templates/html.tpl \
                                -o trivy-reports/trivy-backend-report.html greenx-backend:${BUILD_NUMBER}

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
                                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                                    docker tag greenx-backend:${BUILD_NUMBER} $DOCKER_USER/greenx-backend:${BUILD_NUMBER}
                                    docker tag greenx-frontend:${BUILD_NUMBER} $DOCKER_USER/greenx-frontend:${BUILD_NUMBER}

                                    docker push $DOCKER_USER/greenx-backend:${BUILD_NUMBER}
                                    docker push $DOCKER_USER/greenx-frontend:${BUILD_NUMBER}
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
                sh '''
                    docker compose -f docker-compose.images.yml pull
                    docker compose -f docker-compose.images.yml up -d
                '''
            }
        }
    }
}
