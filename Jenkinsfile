pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dckr_pat_UxrLRYE9UjRyFiznHNsVcM7CCo0')
        DOCKER_IMAGE = 'ummoo/ci-cd-demo-app'
        DOCKER_TAG = "v${BUILD_NUMBER}"
        GITHUB_REPO = 'https://github.com/MyoMyintOoCV/ci-cd-demo-app.git'
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
                to: 'mom27380@gmail.com'
            )
        }
        failure {
            echo 'Pipeline failed!'
            emailext (
                subject: "Pipeline Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "The pipeline failed. Please check: ${env.BUILD_URL}",
                to: 'mom27380@gmail.com'
            )
        }
    }
}
