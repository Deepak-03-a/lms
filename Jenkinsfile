pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                // Checkout the Git repository
                checkout scm
            }
        }
        stage('Build and Tag') {
            steps {
                echo 'Building and Tagging Docker image...'
                // Change to the directory containing the Dockerfile
                sh 'cd webapp'
                // Get the version from package.json
                script {
                    def packageJsonText = readFile 'webapp/package.json'
                    def packageJson = new groovy.json.JsonSlurper().parseText(packageJsonText)
                    env.VERSION = packageJson.version.toString() // Ensure it's a String
                    echo "package.json version: ${env.VERSION}"
                    // Build and tag the Docker image with the version from package.json
                    sh "docker build -t your-dockerhub-username/your-image-name:${VERSION} -f Dockerfile ."
                }
            }
        }
        stage('Push') {
            steps {
                echo "Pushing Docker image..."
                // Login to Docker Hub (or your registry)
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'docker login -u "$DOCKER_USERNAME" -p "$DOCKER_PASSWORD"'
                    // Push the Docker image with the version tag
                    sh "docker push your-dockerhub-username/your-image-name:${VERSION}"
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
