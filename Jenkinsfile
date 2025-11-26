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
                    docker build -t yeshfaandleeb1/greenx-backend:latest \
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
                    docker build -t yeshfaandleeb1/greenx-frontend:latest \
                    ./GreenX_DCS_Assesment_Tool-main/greenX-assessment-tool-frontend
                '''
            }
        }

        /* ================================
           TRIVY + PUSH PARALLEL
        ================================= */
        stage('Scan & Push Images (Parallel Stage)') {
            steps {
                script {
                    parallel(

                        "Trivy Scan": {
                            sh '''
                                mkdir -p trivy-reports

                                trivy image --format template --template @/opt/trivy-templates/html.tpl \
                                -o trivy-reports/backend.html yeshfaandleeb1/greenx-backend:latest

                                trivy image --format template --template @/opt/trivy-templates/html.tpl \
                                -o trivy-reports/frontend.html yeshfaandleeb1/greenx-frontend:latest
                            '''

                            archiveArtifacts artifacts: 'trivy-reports/*.html', fingerprint: true
                        },

                        "Docker Push": {
                            withCredentials([usernamePassword(
                                credentialsId: 'dockerhub-credentials',
                                usernameVariable: 'DOCKER_USER',
                                passwordVariable: 'DOCKER_PASS'
                            )]) {
                                sh '''
                                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                                    docker push yeshfaandleeb1/greenx-backend:latest
                                    docker push yeshfaandleeb1/greenx-frontend:latest
                                '''
                            }
                        }
                    )
                }
            }
        }

        /* ================================
           MANUAL APPROVAL BEFORE DEPLOYMENT
        ================================= */
        stage('Approval Required') {
            steps {
                script {
                    emailext(
                        to: "yeshfaandleeb05@gmail.com",
                        subject: "APPROVAL REQUIRED: GreenX Pipeline Build #${BUILD_NUMBER}",
                        body: """\
Hello,

The pipeline is waiting for manual approval before deployment.

Click to approve:
${env.BUILD_URL}

Thanks,
Jenkins System
"""
                    )

                    timeout(time: 30, unit: 'MINUTES') {
                        input message: "Approve deployment to production?", ok: "DEPLOY NOW"
                    }
                }
            }
        }

        /* ================================
           DEPLOY USING DOCKER COMPOSE
        ================================= */
        stage('Deploy Using Docker Compose') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "===== Pulling Latest Images ====="
                        docker compose -f docker-compose.images.yml pull

                        echo "===== Starting Deployment ====="
                        docker compose -f docker-compose.images.yml up -d

                        echo "===== Deployment Completed ====="
                    '''
                }
            }
        }
    }
}
