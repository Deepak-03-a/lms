pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building Docker image...'
                // Change to the directory containing the Dockerfile
                sh 'cd webapp'
                // Check for Dockerfile
                sh 'docker --version'  // Corrected line
                sh 'ls -l'
                // Build the Docker image, tagging it with the version from package.json
                sh 'docker build -t your-dockerhub-deepak13333/lms-test:${VERSION} -f .'
                script {
                    def packageJson = readJSON file: 'webapp/package.json'
                    env.VERSION = packageJson.version;
                    echo "Docker Image Version: ${env.VERSION}"
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
                    sh 'docker push your-dockerhub-deepak13333/lms-test:${VERSION}'
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