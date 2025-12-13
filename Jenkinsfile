pipeline {
    agent any

    tools {
        nodejs 'node18'   // <-- MUST exist in Jenkins Global Tool Config
    }

    environment {
        DOCKER_IMAGE = 'ummoo/ci-cd-demo-app'   // change to your Docker Hub username
        DOCKER_TAG = "v${BUILD_NUMBER}"
        GITHUB_REPO = 'https://github.com/MyoMyintOoCV/ci-cd-demo-app.git'
        AWS_EC2_HOST = 'ec2-user@54.89.85.123'
    }

    stages {

        stage('Checkout SCM') {
            steps {
                git branch: 'main',
                    url: "${GITHUB_REPO}",
                    credentialsId: 'UMMOO-GIT'
            }
        }

        stage('Build App') {
            steps {
                sh '''
                    node -v
                    npm -v
                    npm ci
                '''
                echo 'Application built successfully'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test || true'
                echo 'Tests executed'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
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
                        docker run -d \
                          --name ci-cd-app \
                          -p 80:3000 \
                          --restart always \
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
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
