stage('Scan & Push Images (Parallel Stage)') {
    steps {
        script {
            parallel(
                "Trivy Image Scan": {
                    sh '''
                        mkdir -p trivy-reports

                        echo "=== Trivy Scan: Backend ==="
                        trivy image --format html --output trivy-reports/trivy-vote-report.html greenx-backend:${BUILD_NUMBER}

                        echo "=== Trivy Scan: Frontend ==="
                        trivy image --format html --output trivy-reports/trivy-result-report.html greenx-frontend:${BUILD_NUMBER}

                        echo "=== Trivy Scan: Worker ==="
                        trivy image --format html --output trivy-reports/trivy-worker-report.html greenx-worker:${BUILD_NUMBER}
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

                            docker tag greenx-backend:${BUILD_NUMBER} aleemdevops/greenx-backend:${BUILD_NUMBER}
                            docker tag greenx-frontend:${BUILD_NUMBER} aleemdevops/greenx-frontend:${BUILD_NUMBER}
                            docker tag greenx-worker:${BUILD_NUMBER} aleemdevops/greenx-worker:${BUILD_NUMBER}

                            docker push aleemdevops/greenx-backend:${BUILD_NUMBER}
                            docker push aleemdevops/greenx-frontend:${BUILD_NUMBER}
                            docker push aleemdevops/greenx-worker:${BUILD_NUMBER}
                        '''
                    }
                }
            )
        }
    }
}
