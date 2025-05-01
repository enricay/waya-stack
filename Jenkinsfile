pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        DOCKER_IMAGE = "<your-dockerhub-username>/simple-web-app"
    }
    stages {
        stage('Clone Repository') {
            steps {
                git credentialsId: 'github', url: 'https://github.com/<your-username>/simple-web-app.git', branch: 'main'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('Deploy with Docker Compose') {
            steps {
                sh 'docker-compose -f docker-compose.yml down'
                sh 'docker-compose -f docker-compose.yml up -d'
            }
        }
    }
}