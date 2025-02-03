pipeline {
    agent any

    environment {
        PATH = "C:\\Program Files\\Docker\\Docker\\resources\\bin;C:\\Windows\\System32"
        DOCKER_HOST = 'tcp://localhost:2375'
        IBM_CLOUD_REGISTRY_NAMESPACE = 'config-manage'
        IBM_CLOUD_REGISTRY_URL = 'jp.icr.io'  // IBM Container Registry for Chennai
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
                    bat """
                    echo 🔨 Building Docker Image...
                    "C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe" build -t config-manage:latest -f docker/Dockerfile .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    bat 'echo 📦 Preparing to push Docker image...'
                     withCredentials([string(credentialsId: 'ibm-cloud-api-key', variable: 'IBM_CLOUD_API_KEY')]) {

                    // Use Trusted Profile Login Instead of API Key
                    bat """
                    echo 🔐 Logging into IBM Cloud with Trusted Profile...
                    "C:\\Program Files\\IBM\\Cloud\\bin\\ibmcloud.exe" login --apikey %IBM_CLOUD_API_KEY%  -r in-che
                    "C:\\Program Files\\IBM\\Cloud\\bin\\ibmcloud.exe" cr login
                    """}

                    // Tag and Push Docker Image
                    def imageName = "${IBM_CLOUD_REGISTRY_URL}/${IBM_CLOUD_REGISTRY_NAMESPACE}/config-manage:latest"
                    bat """
                    echo 🚀 Tagging and Pushing Docker Image...
                    "C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe" tag config-manage:latest ${imageName}
                    "C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe" push ${imageName}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    bat """
                    echo ⚙️ Configuring Kubernetes Cluster...
                    "C:\\Program Files\\IBM\\Cloud\\bin\\ibmcloud.exe" ks cluster config --cluster <your-cluster-name>

                    echo 📡 Deploying to Kubernetes...
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                    """
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
