pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        DOCKER_IMAGE = "enricay/waya-stack"
        AWS_CREDENTIALS = credentials('aws-eks-credentials')
    }
    stages {
        stage('Clone Repository') {
            steps {
                git credentialsId: 'github', url: 'https://github.com/enricay/waya-stack.git', branch: 'main'
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
        // stage('Deploy to Kubernetes') {
        //     steps {
        //         withKubeConfig([credentialsId: 'kubeconfig']) {
        //             sh 'kubectl apply -f k8s-deployment.yml'
        //         }
        //     }
        // }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-eks-credentials']]) {
                    script {
                        // Configure AWS credentials for EKS
                        sh '''
                            aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                            aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                            aws configure set region eu-west-1
                        '''
                        // Update kubeconfig to authenticate with EKS
                        sh '''
                            aws eks update-kubeconfig --region eu-west-1 --name capplc
                        '''
                        // Apply Kubernetes manifest
                        sh '''
                            kubectl get ns
                        '''
                    }
                }
            }
        }

    }
}