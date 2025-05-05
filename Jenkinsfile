pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                // Checkout the Git repository
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'Building Docker image...'
                // Change to the directory containing the Dockerfile
                sh 'cd webapp'
                // Print current directory
                sh 'pwd'
                // List files in webapp
                sh 'ls -l'
                // Build the Docker image, tagging it with the version from package.json
                script {
                    def packageJson = readJSON file: 'package.json'
                    env.VERSION = packageJson.version
                    echo "package.json version: ${env.VERSION}"
                    sh "docker build -t deepak13333/lms-test:${VERSION} -f Dockerfile ."
                }
            }
        }
        stage('Push') {
            steps {
                echo "Pushing Docker image..."
                // Login to Docker Hub (or your registry)
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'docker login -u "$DOCKER_USERNAME" -p "$DOCKER_PASSWORD"'
                    // Push the Docker image
                    sh 'docker push deepak13333/lms-test:${VERSION}'
                }
            }
        }
        stage('Clean Up Workspace') {
            steps {
                echo 'Cleaning Work Space'
                // Install Cleanup Workspace plugin to make below command work
                cleanWs()
            }
        }
    }
}
