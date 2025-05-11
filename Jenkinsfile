pipeline {
    agent { label 'local-vm' }

    parameters {
        string(name: 'DOCKERHUB_CREDENTIALS_ID',
               defaultValue: 'dockerhub-credentials',
               description: 'ID of the Docker Hub credentials in Jenkins')
        string(name: 'DOCKERHUB_USERNAME',
               defaultValue: 'your-dockerhub-username',
               description: 'Your Docker Hub username')
        string(name: 'IMAGE_NAME',
               defaultValue: 'your-dockerhub-username/lms-frontend',
               description: 'Name of the Docker image in Docker Hub')
        string(name: 'MINIKUBE_CONTEXT',
               defaultValue: 'minikube',
               description: 'Name of the Minikube context')
        string(name: 'NODE_PORT',
               defaultValue: '30000',
               description: 'The NodePort to expose the service on (requested)')
    }

    stages {
        stage('Version') {
            steps {
                script {
                    def packageJson = readJSON file: 'webapp/package.json'
                    env.VERSION = packageJson.version
                    echo "Version from package.json: ${env.VERSION}"
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    echo "Logging into Docker Hub..."
                    dockerLogin(credentialsId: params.DOCKERHUB_CREDENTIALS_ID,
                                username: params.DOCKERHUB_USERNAME)

                    echo "Building Docker image..."
                    def dockerImage = docker.build(image: "${params.IMAGE_NAME}:${env.VERSION}", dir: 'webapp')

                    echo "Pushing Docker image tags..."
                    dockerImage.push("${env.VERSION}")
                    dockerImage.push('latest')
                    env.DOCKER_IMAGE_TAGGED = "${params.IMAGE_NAME}:${env.VERSION}"
                    echo "Docker image pushed: ${env.DOCKER_IMAGE_TAGGED}"
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    echo "Setting Minikube context: ${params.MINIKUBE_CONTEXT}"
                    sh "kubectl config use-context ${params.MINIKUBE_CONTEXT}"

                    echo "Applying Kubernetes Deployment and Service..."
                    def deploymentYaml = """
apiVersion: apps/v1
kind: Deployment
metadata:
  name: lms-fe
spec:
  replicas: 1
  selector:
    matchLabels:
      app: lms-fe
  template:
    metadata:
      labels:
        app: lms-fe
    spec:
      containers:
        - name: frontend-container
          image: ${env.DOCKER_IMAGE_TAGGED}
          imagePullPolicy: Always
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: lms-fe-service
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      protocol: TCP
      nodePort: ${params.NODE_PORT}
  selector:
    app: lms-fe
"""
                    sh "kubectl apply -f -" << deploymentYaml
                    echo "LMS Frontend Deployed to Minikube!"

                    env.MINIKUBE_IP = sh(returnStdout: true, script: "minikube ip").trim()
                    echo "Application is accessible at: http://${env.MINIKUBE_IP}:${params.NODE_PORT} (Check NodePort with 'kubectl get service lms-fe-service -o wide')"
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
