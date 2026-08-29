pipeline {
    agent any

    environment {
        AWS_REGION     = 'ap-south-1'
        AWS_ACCOUNT_ID = '610269527042'

        BACKEND_IMAGE  = 'three-tier/backend'
        FRONTEND_IMAGE = 'three-tier/frontend'
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
                    docker build -t three-tier-backend:latest .
                    '''
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('Application-Code/frontend') {
                    sh '''
                    docker build -t three-tier-frontend:latest .
                    '''
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image --severity HIGH,CRITICAL three-tier-backend:latest
                trivy image --severity HIGH,CRITICAL three-tier-frontend:latest
                '''
            }
        }

        stage('ECR Login') {
            steps {
                sh '''
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS \
                --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Tag Images') {
            steps {
                sh '''
                docker tag three-tier-backend:latest \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_IMAGE}:${BUILD_NUMBER}

                docker tag three-tier-frontend:latest \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${FRONTEND_IMAGE}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Backend Image') {
            steps {
                sh '''
                docker push \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_IMAGE}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Push Frontend Image') {
            steps {
                sh '''
                docker push \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${FRONTEND_IMAGE}:${BUILD_NUMBER}
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
