pipeline {
    agent any

    environment {
        // Replace with your Docker Hub username and repository name
        DOCKER_IMAGE = 'roopar2567/ci-cd-finexo'
        // Jenkins credentials ID for Docker Hub
        DOCKER_CREDENTIALS_ID = 'dockerHub'
        INITIAL_PORT = '4000'
    }

    stages {
        stage('Clone Repository') {
            steps {
                // Clone the GitHub repository
                git branch: 'main', url: 'https://github.com/Roopa-R256/ci-cd-finexo.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    dockerImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Test Port') {
            steps {
                script {
                    def testPort = env.INITIAL_PORT as Integer
                    def containerId = ''

                    while (true) {
                        try {
                            // Try to run the Docker container on the current port
                            containerId = sh(script: "docker run -d -p ${testPort}:80 ${DOCKER_IMAGE}:${env.BUILD_NUMBER}", returnStdout: true).trim()

                            // Wait for the container to start
                            sleep 10

                            // Test if the application responds on the desired port
                            sh "curl -f http://localhost:${testPort} || (echo 'Application is not running on port ${testPort}' && exit 1)"
                            echo "Application is running successfully on port ${testPort}"

                            // If successful, break out of the loop
                            break
                        } catch (Exception e) {
                            echo "Port ${testPort} is already in use. Incrementing port by 10."
                            testPort += 10

                            // Stop and remove any container started before the failure
                            if (containerId) {
                                sh "docker stop ${containerId}"
                                sh "docker rm ${containerId}"
                            }
                        }
                    }

                    // Stop and remove the container after testing
                    if (containerId) {
                        sh "docker stop ${containerId}"
                        sh "docker rm ${containerId}"
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    // Log in to Docker Hub and push the image
                    docker.withRegistry('', DOCKER_CREDENTIALS_ID) {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
        
        stage('Clean Up') {
            steps {
                script {
                    // Remove the Docker image from the local environment
                    sh "docker rmi ${DOCKER_IMAGE}:${env.BUILD_NUMBER}"
                    sh "docker rmi ${DOCKER_IMAGE}:latest"
                }
            }
        }
    }

    post {
        success {
            echo 'Docker image built and pushed successfully.'
        }
        failure {
            echo 'An error occurred during the pipeline execution.'
        }
    }
}
