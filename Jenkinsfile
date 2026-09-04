pipeline {
    agent any

    environment {
        AWS_REGION     = 'ap-south-1'
        AWS_ACCOUNT_ID = '610269527042'

        BACKEND_IMAGE  = 'three-tier/backend'
        FRONTEND_IMAGE = 'three-tier/frontend'

        GITOPS_REPO    = 'https://github.com/ameya018/Three-tier-gitops.git'
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

        stage('ECR Login') {
            steps {
                sh '''
                aws ecr get-login-password --region ${AWS_REGION} | \
                docker login --username AWS \
                --password-stdin \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Build Backend Image') {
            steps {
                dir('Application-Code/backend') {
                    sh '''
                    docker build \
                    -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_IMAGE}:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('Application-Code/frontend') {
                    sh '''
                    docker build \
                    -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${FRONTEND_IMAGE}:${BUILD_NUMBER} .
                    '''
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh '''
                trivy image --severity HIGH,CRITICAL \
                ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_IMAGE}:${BUILD_NUMBER}

                trivy image --severity HIGH,CRITICAL \
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

        stage('Update GitOps Repo') {
            steps {

                withCredentials([
                    string(
                        credentialsId: 'github-token',
                        variable: 'GITHUB_TOKEN'
                    )
                ]) {

                    sh '''
                    rm -rf gitops

                    git clone https://${GITHUB_TOKEN}@github.com/ameya018/Three-tier-gitops.git gitops

                    sed -i "s|image: .*three-tier/backend:.*|image: ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${BACKEND_IMAGE}:${BUILD_NUMBER}|g" \
                    gitops/backend/deployment.yaml

                    sed -i "s|image: .*three-tier/frontend:.*|image: ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${FRONTEND_IMAGE}:${BUILD_NUMBER}|g" \
                    gitops/frontend/deployment.yaml

                    cd gitops

                    git config user.email "jenkins@local"
                    git config user.name "Jenkins"
		    
                    git remote set-url origin https://${GITHUB_TOKEN}@github.com/ameya018/Three-tier-gitops.git

                    git add .

                    git commit -m "Update image tags to build ${BUILD_NUMBER}" || true

                    git push origin HEAD:main
                    '''
                }
            }
        }
    }

    post {

        always {
            sh '''
            docker images
            '''
        }

        success {
            echo 'CI/CD Pipeline Completed Successfully'
        }

        failure {
            echo 'CI/CD Pipeline Failed'
        }
    }
}
