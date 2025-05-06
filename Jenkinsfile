pipeline {
    agent any

    parameters {
        string(name: 'DOCKERHUB_CREDENTIALS_ID',
               defaultValue: 'dockerhub-credentials',  // Use the ID you provided
               description: 'ID of the Docker Hub credentials in Jenkins')
        string(name: 'DOCKERHUB_USERNAME',
               defaultValue: 'your-dockerhub-username',
               description: 'Your Docker Hub username')
        string(name: 'IMAGE_NAME',
               defaultValue: 'your-dockerhub-username/lms-frontend',
               description: 'Name of the Docker image in Docker Hub')
    }

    stages {
        stage('Checkout') {
            steps {
                git(credentialsId: 'github-credentials-id',
                    url: 'https://github.com/Deepak-03-a/lms.git',
                    branch: 'project-6')
            }
        }

        stage('Version and Tag') {
            steps {
                script {
                    def packageJson = readJSON file: 'webapp/package.json'
                    env.VERSION = packageJson.version
                    echo "Version from package.json: ${env.VERSION}"

                    sh "git tag ${env.VERSION}"
                    sh "git push --tags"
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    dockerLogin(credentialsId: params.DOCKERHUB_CREDENTIALS_ID,
                                username: params.DOCKERHUB_USERNAME)  // Include username

                    def dockerImage = docker.build("${params.IMAGE_NAME}:${env.VERSION}", dir: 'webapp')
                    dockerImage.push()
                    dockerImage.push('latest')
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Deploying image ${params.IMAGE_NAME}:${env.VERSION} using port mapping"
                    //  Run the Docker container with hardcoded port mapping.
                    sh "docker run -d -p 80:80 ${params.IMAGE_NAME}:${env.VERSION}"
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
