pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "three-tier-backend"
        FRONTEND_IMAGE = "three-tier-frontend"
    }

    tools {
        sonarQube 'sonarqube'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Workspace Info') {
            steps {
                sh '''
                pwd
                ls -la
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    sonar-scanner \
                    -Dsonar.projectKey=three-tier-devsecops \
                    -Dsonar.projectName=three-tier-devsecops \
                    -Dsonar.sources=Application-Code
                    '''
                }
            }
        }

        stage('Trivy Filesystem Scan') {
            steps {
                sh '''
                trivy fs --severity HIGH,CRITICAL .
                '''
            }
        }

        stage('Build Backend Image') {
            steps {
                dir('Application-Code/backend') {
                    sh '''
                    docker build -t ${BACKEND_IMAGE}:latest .
                    '''
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('Application-Code/frontend') {
                    sh '''
                    docker build -t ${FRONTEND_IMAGE}:latest .
                    '''
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image --severity HIGH,CRITICAL ${BACKEND_IMAGE}:latest
                trivy image --severity HIGH,CRITICAL ${FRONTEND_IMAGE}:latest
                '''
            }
        }
    }

    post {
        always {
            sh 'docker images'
        }

        success {
            echo 'Pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed'
        }
    }
}
