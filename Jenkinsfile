pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        DOCKER_IMAGE = "enricay/waya-stack"
        AWS_CREDENTIALS = credentials('aws-eks-credentials')
        OCTOPUS_API_KEY = credentials('octopus-api-key')  // Store your Octopus API key in Jenkins Credentials
        OCTOPUS_SERVER = 'https://waya.octopus.app'  // Replace with your Octopus server URL
        OCTOPUS_PROJECT = 'waya-stack'  // Replace with your Octopus project name
        OCTOPUS_ENVIRONMENT = 'Development'  // Replace with the environment you want to deploy to
        OCTOPUS_RELEASE_VERSION = "0.0.1"  // Set the release version
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
        stage('Trigger Octopus Deploy') {
            steps {
                script {
                    // Trigger Octopus Deployment
                    sh '''
                    curl -X POST ${OCTOPUS_SERVER}/api/deployments \
                        -H "X-Octopus-ApiKey: ${OCTOPUS_API_KEY}" \
                        -H "Content-Type: application/json" \
                        -d '{
                            "ProjectId": "${OCTOPUS_PROJECT}",
                            "EnvironmentId": "${OCTOPUS_ENVIRONMENT}",
                            "ReleaseVersion": "${OCTOPUS_RELEASE_VERSION}"
                        }'
                    '''
                }
            }
        }
    }
}
