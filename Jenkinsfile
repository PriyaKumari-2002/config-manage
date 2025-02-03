pipeline {
    agent any

    environment {
        PATH = "C:\\Program Files\\Docker\\Docker\\resources\\bin;C:\\Windows\\System32"  // Added Docker to PATH
        DOCKER_HOST = 'tcp://localhost:2375'  // Expose Docker daemon if needed
        IBM_CLOUD_API_KEY = credentials('ibmcloud-api-key')
        IBM_CLOUD_REGISTRY_NAMESPACE = 'config-manage'
        IBM_CLOUD_REGISTRY_URL = 'jp.icr.io'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/PriyaKumari-2002/config-manage.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe" build -t config-manage:latest --file=docker/Dockerfile .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    bat 'echo Hello'
                    
                    // IBM Cloud Login
                    bat 'ibmcloud login --apikey %IBM_CLOUD_API_KEY% -r us-south'
                    bat 'ibmcloud cr login'

                    // Tag and push the Docker image
                    def imageName = "${IBM_CLOUD_REGISTRY_URL}/${IBM_CLOUD_REGISTRY_NAMESPACE}/config-manage:latest"
                    bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe" tag config-manage:latest %imageName%'
                    bat '"C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe" push %imageName%'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    bat 'ibmcloud ks cluster config --cluster <your-cluster-name>'
                    bat 'kubectl apply -f k8s/deployment.yaml'
                    bat 'kubectl apply -f k8s/service.yaml'
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
