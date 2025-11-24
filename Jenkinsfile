pipeline {
    agent any

    tools {
        // SonarQube Scanner installed from Jenkins -> Global Tool Configuration
        sonarRunner 'DefaultScanner'
    }

    environment {
        // SonarQube token stored in Jenkins Credentials as "sonar-token"
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Local') {
                    sh '''
/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/DefaultScanner/bin/sonar-scanner \
  -Dsonar.login=$SONAR_TOKEN \
  -Dsonar.ws.timeout=600
'''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Smoke Test') {
            when {
                expression { return false }  // Enable later if needed
            }
            steps {
                echo "Docker stage will be added later."
            }
        }
    }
}
