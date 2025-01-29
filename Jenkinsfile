pipeline {
    agent any

    environment {
        DOCKER_HOST = 'tcp://localhost:2375'  // Use the Docker API over TCP if required
        // IBM Cloud Container Registry credentials
        IBM_CLOUD_API_KEY = credentials('ibmcloud-api-key')
        IBM_CLOUD_REGISTRY_NAMESPACE = 'config-manage'
        IBM_CLOUD_REGISTRY_URL = 'jp.icr.io'
    }

    stages {
        // Stage 1: Checkout code from GitHub
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/PriyaKumari-2002/config-manage.git'
            }
        }

        // Stage 2: Build Docker image
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("config-manage:latest", "-f docker/Dockerfile .")
                }
            }
        }

        // Stage 3: Push Docker image to IBM Cloud Container Registry
        stage('Push Docker Image') {
            steps {
                script {
                    // Authenticate with IBM Cloud
                    bat "ibmcloud login --apikey $IBM_CLOUD_API_KEY"
                    bat "ibmcloud cr login"

                    // Tag and push the Docker image
                    docker.withRegistry("https://${IBM_CLOUD_REGISTRY_URL}", "ibm-cloud-credentials") {
                        docker.image("config-manage:latest").push("${IBM_CLOUD_REGISTRY_URL}/${IBM_CLOUD_REGISTRY_NAMESPACE}/config-manage:latest")
                    }
                }
            }
        }

        // Stage 4: Deploy to Kubernetes
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Apply Kubernetes manifests
                    bat "kubectl apply -f k8s/deployment.yaml"
                    bat "kubectl apply -f k8s/service.yaml"
                }
            }
        }
    }

    // Post-build actions
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
} 
