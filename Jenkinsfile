pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('DockerHub')
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Clone the repository
                    checkout scm
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    // Build steps (replace with your build commands)
                    sh 'echo "Building..."'
                    sh 'ls'
                    sh 'docker images'
                    sh 'docker build -t sampleapp .'
                    sh 'docker images'
                }
            }
        }

        stage('Push image') {
            steps {
                script {
                    // Log in to Docker Hub using Jenkins credentials
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerHub', usernameVariable: 'DOCKER_HUB_CREDENTIALS_USR', passwordVariable: 'DOCKER_HUB_CREDENTIALS_PSW']]) {
                        sh 'docker login -u $DOCKER_HUB_CREDENTIALS_USR -p $DOCKER_HUB_CREDENTIALS_PSW'
                    }
                    // Push the image to Docker Hub
                    sh 'docker push gifti/ci_cd:latest'
                }
            }
        }

        stage('Pull Docker Image') {
            steps {
                script {
                    // Pull the image from Docker Hub
                    sh 'docker pull gifti/ci_cd:latest'
                    sh 'docker images gifti/ci_cd:latest'

                    // Run Docker container in the background and capture the container ID
                    def containerId = sh(script: 'docker run -d -p 4000:5000 sampleapp', returnStdout: true).trim()

                    // Save container ID for later
                    env.CONTAINER_ID = containerId

                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // Wait for a few seconds to ensure the container is fully started
                    sleep time: 10, unit: 'SECONDS'

                    // Run tests using curl against the running container
                    def testOutput = sh(script: 'curl http://34.201.130.141:4000', returnStdout: true).trim()

                    // Check if the output is 'Hello, World!'
                    if (testOutput == 'Hello, World!') {
                        echo 'Test passed: Output is Hello, World!'
                    } else {
                        error 'Test failed: Output is not Hello, World!'
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    // Stop and remove the Docker container after tests
                    sh "docker stop ${env.CONTAINER_ID}"
                    sh "docker rm ${env.CONTAINER_ID}"
                }
            }
        }
    }
}

