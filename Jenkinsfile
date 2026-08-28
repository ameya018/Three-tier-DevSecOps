pipeline {
    agent any

    environment {
        FRONTEND_IMAGE = "three-tier-frontend"
        BACKEND_IMAGE  = "three-tier-backend"
        IMAGE_TAG      = "${BUILD_NUMBER}"
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
                    ls -la Application-Code
                '''
            }
        }

        stage('Build Backend Image') {
            steps {
                dir('Application-Code/backend') {
                    sh '''
                        docker build \
                        -t ${BACKEND_IMAGE}:${IMAGE_TAG} .
                    '''
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('Application-Code/frontend') {
                    sh '''
                        docker build \
                        -t ${FRONTEND_IMAGE}:${IMAGE_TAG} .
                    '''
                }
            }
        }

        stage('Verify Images') {
            steps {
                sh '''
                    docker images | grep three-tier
                '''
            }
        }
    }

    post {
        success {
            echo 'Build completed successfully'
        }

        failure {
            echo 'Build failed'
        }

        always {
            cleanWs()
        }
    }
}
