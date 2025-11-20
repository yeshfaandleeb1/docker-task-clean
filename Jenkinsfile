pipeline {
    agent any

    environment {
        SONAR_SCANNER_HOME = "/opt/sonar-scanner"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        url: 'https://github.com/yeshfaandleeb1/docker-task-clean.git',
                        credentialsId: 'github_creds'
                    ]]
                ])
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonar') {
                    withCredentials([string(credentialsId: 'SONAR_TOKENN', variable: 'SONAR_LOGIN')]) {
                        sh """
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=GreenX \
                        -Dsonar.projectName=GreenX \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://3.239.85.144:9000 \
                        -Dsonar.login=${SONAR_LOGIN}
                        """
                    }
                }
            }
        }

        stage('Sonar Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true, credentialsId: 'QUALITY_GATE_TOKEN'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t greenx-backend ./GreenX_DCS_Assesment_Tool_Backend'
                sh 'docker build -t greenx-frontend ./greenX-assessment-tool-frontend'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub_creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                    echo $PASS | docker login -u $USER --password-stdin
                    docker tag greenx-backend $USER/greenx-backend:latest
                    docker tag greenx-frontend $USER/greenx-frontend:latest
                    docker push $USER/greenx-backend:latest
                    docker push $USER/greenx-frontend:latest
                    """
                }
            }
        }

        stage('List Docker Images') {
            steps {
                sh 'docker images'
            }
        }
    }

    post {
        always {
            emailext(
                subject: "Jenkins Pipeline Status: ${currentBuild.currentResult}",
                body: "Build result: ${currentBuild.currentResult}",
                to: "yeshfaandleeb05@gmail.com"
            )
        }
    }
}
