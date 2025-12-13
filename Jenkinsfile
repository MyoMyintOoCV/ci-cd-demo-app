pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')
        DOCKER_IMAGE = 'your-dockerhub-username/ci-cd-demo-app'
        DOCKER_TAG = "v${BUILD_NUMBER}"
        GITHUB_REPO = 'https://github.com/your-username/ci-cd-demo-app.git'
        AWS_EC2_HOST = 'ec2-user@your-ec2-public-ip'
        AWS_SSH_KEY = credentials('aws-ec2-ssh-key')
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: env.GITHUB_REPO
            }
        }
        
        stage('Build App') {
            steps {
                sh 'npm ci'
                echo 'Application built successfully'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'npm test'
                junit 'reports/**/*.xml'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }
        
        stage('Push to Private Registry') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                    }
                }
            }
        }
        
        stage('Deploy to AWS EC2') {
            steps {
                sshagent(['aws-ec2-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ${AWS_EC2_HOST} '
                            docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker stop ci-cd-app || true
                            docker rm ci-cd-app || true
                            docker run -d \\
                                --name ci-cd-app \\
                                -p 80:3000 \\
                                --restart always \\
                                ${DOCKER_IMAGE}:${DOCKER_TAG}
                        '
                    """
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
            emailext (
                subject: "Pipeline Success: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "The pipeline completed successfully.\n\nBuild: ${env.BUILD_URL}",
                to: 'your-email@example.com'
            )
        }
        failure {
            echo 'Pipeline failed!'
            emailext (
                subject: "Pipeline Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "The pipeline failed. Please check: ${env.BUILD_URL}",
                to: 'your-email@example.com'
            )
        }
    }
}
EOF

# Create multiple pipeline scripts for different environments
mkdir -p jenkins/scripts

# Development pipeline
cat > jenkins/Jenkinsfile.dev << 'EOF'
pipeline {
    agent any
    stages {
        stage('Dev Build & Test') {
            steps {
                sh 'npm ci'
                sh 'npm test'
                sh 'docker build -t app:dev .'
            }
        }
    }
}
EOF

# Staging pipeline
cat > jenkins/Jenkinsfile.staging << 'EOF'
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'your-dockerhub-username/app-staging'
    }
    stages {
        stage('Staging Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:staging .'
                sh 'docker push ${DOCKER_IMAGE}:staging'
            }
        }
    }
}
EOF

# Production pipeline
cat > jenkins/Jenkinsfile.prod << 'EOF'
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'your-dockerhub-username/app-prod'
    }
    stages {
        stage('Production Build') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:prod .'
                sh 'docker push ${DOCKER_IMAGE}:prod'
            }
        }
        stage('Deploy to Production') {
            steps {
                sh '''
                    ssh user@production-server "
                        docker pull ${DOCKER_IMAGE}:prod
                        docker-compose up -d
                    "
                '''
            }
        }
    }
}