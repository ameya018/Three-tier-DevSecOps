pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"
        AWS_ACCOUNT_ID = "610269527042"

        BACKEND_IMAGE  = "three-tier-backend"
        FRONTEND_IMAGE = "three-tier-frontend"

        BACKEND_REPO   = "three-tier-backend"
        FRONTEND_REPO  = "three-tier-frontend"

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
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner'

                    withSonarQubeEnv('sonarqube') {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=three-tier-devsecops \
                        -Dsonar.projectName=three-tier-devsecops \
                        -Dsonar.sources=Application-Code
                        """
                    }
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

        stage('ECR Login') {
            steps {
                sh '''
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login \
                --username AWS \
                --password-stdin \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Tag Images') {
            steps {
                sh '''
                docker tag ${BACKEND_IMAGE}:latest \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_REPO}:${IMAGE_TAG}

                docker tag ${FRONTEND_IMAGE}:latest \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${FRONTEND_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Backend Image') {
            steps {
                sh '''
                docker push \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_REPO}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Frontend Image') {
            steps {
                sh '''
                docker push \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${FRONTEND_REPO}:${IMAGE_TAG}
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
