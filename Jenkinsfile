pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        DOCKER_IMAGE = "enricay/waya-stack"
        OCTOPUS_SERVER = 'https://waya.octopus.app'  // Octopus server URL
        OCTOPUS_PROJECT = 'waya-stack'  // Octopus project name
        OCTOPUS_ENVIRONMENT = 'Development'  // Environment to deploy to
        OCTOPUS_RELEASE_VERSION = "0.0.1"  // Release version from Octopus
        OCTOPUS_API_KEY = credentials('octopus-api-key') // Octopus API key from Jenkins credentials
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
        stage('Get ReleaseId from Octopus') {
            steps {
                script {
                    // Fetch the release information from Octopus using the Octopus API
                    def releaseResponse = sh(script: """
                        curl -X GET ${OCTOPUS_SERVER}/api/Spaces-1/projects/Projects-1/releases \
                            -H "X-Octopus-ApiKey: \$OCTOPUS_API_KEY" \
                            -H "Content-Type: application/json"
                    """, returnStdout: true).trim()

                    // Debugging step: print the response to check
                    echo "Response from Octopus: ${releaseResponse}"

                    // Parse the response to get the ReleaseId based on the release version
                    def releaseId = ''
                    if (releaseResponse != '') {
                        releaseId = sh(script: """
                            echo '${releaseResponse}' | jq -r '.Items[] | select(.Version == "${OCTOPUS_RELEASE_VERSION}") | .Id'
                        """, returnStdout: true).trim()
                    }

                    // Check if releaseId was found
                    if (releaseId) {
                        env.OCTOPUS_RELEASE_ID = releaseId
                    } else {
                        error "Release version ${OCTOPUS_RELEASE_VERSION} not found in Octopus"
                    }
                }
            }
        }
        stage('Trigger Octopus Deploy') {
            steps {
                script {
                    // Trigger the deployment to Octopus using the ReleaseId and environment
                    sh '''
                    curl -X POST ${OCTOPUS_SERVER}/api/Spaces-1/deployments \
                        -H "X-Octopus-ApiKey: ${OCTOPUS_API_KEY}" \
                        -H "Content-Type: application/json" \
                        -d '{
                            "ProjectId": "${OCTOPUS_PROJECT}",
                            "EnvironmentId": "${OCTOPUS_ENVIRONMENT}",
                            "ReleaseId": "${OCTOPUS_RELEASE_ID}"
                        }'
                    '''
                }
            }
        }
    }
}
