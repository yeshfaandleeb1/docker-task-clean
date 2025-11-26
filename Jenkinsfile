pipeline {
    agent any

    environment {
        SONAR_SCANNER_HOME = tool 'DefaultScanner'
    }

    stages {

        /* ===============================
           CHECKOUT
        ================================= */
        stage('Checkout') {
            steps {
                git url: 'https://github.com/yeshfaandleeb1/docker-task-clean.git', branch: 'main'
            }
        }

        /* ===============================
           SONAR ANALYSIS
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

        /* ===============================
           SONAR QUALITY GATE
        ================================= */
        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        /* ===============================================
           MANUAL APPROVAL + EMAIL NOTIFICATION STAGE
        =============================================== */
        stage('Manual Approval Before Deployment') {
            steps {
                script {

                    // Send approval email
                    emailext(
                        subject: "APPROVAL REQUIRED: Docker-Task Pipeline Build #${env.BUILD_NUMBER}",
                        body: """
Hello Team,

A new build is waiting for your approval.

Pipeline Name: GreenX-CICD-Pipeline  
Build Number: #${env.BUILD_NUMBER}  
Action Needed: APPROVAL REQUIRED

Open Jenkins to approve deployment:
➡ http://192.168.1.36:8080/job/GreenX-CICD-Pipeline/${env.BUILD_NUMBER}/

Regards,
Jenkins Automation
""",
                        to: "yeshfaandleeb05@gmail.com"
                    )

                    // Wait for manual approval
                    timeout(time: 60, unit: 'MINUTES') {
                        input message: """
⚠️  PRODUCTION APPROVAL REQUIRED

Build Number: #${env.BUILD_NUMBER}
Do you want to continue with Docker Build, Trivy Scan, and Deployment?
""",
                        ok: "Approve & Continue"
                    }
                }
            }
        }

        /* ===============================
           DOCKER BUILD BACKEND
        ================================= */
        stage('Docker Build Backend') {
            steps {
                sh '''
                    docker build -t greenx-backend:${BUILD_NUMBER} \
                    ./GreenX_DCS_Assesment_Tool-main/GreenX_DCS_Assesment_Tool_Backend
                '''
            }
        }

        /* ===============================
           DOCKER BUILD FRONTEND
        ================================= */
        stage('Docker Build Frontend') {
            steps {
                sh '''
                    docker build -t greenx-frontend:${BUILD_NUMBER} \
                    ./GreenX_DCS_Assesment_Tool-main/greenX-assessment-tool-frontend
                '''
            }
        }

        /* ===============================
           TRIVY SCAN + PUSH
        ================================= */
        stage('Scan & Push Images (Parallel Stage)') {
            steps {
                script {
                    parallel(

                        /* TRIVY SCAN */
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

                        /* DOCKER PUSH */
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
    }
}
