pipeline {
    agent any

    environment {
        PATH = "C:\\Program Files\\Docker\\Docker\\resources\\bin;C:\\Program Files\\IBM\\Cloud\\bin;C:\\Windows\\System32"
        IBM_CLOUD_REGISTRY_NAMESPACE = 'config-manage'
        IBM_CLOUD_REGISTRY_URL = 'jp.icr.io'  // IBM Container Registry for Tokyo
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
                    docker build -t config-manage:latest -f docker/Dockerfile .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    bat 'echo 📦 Preparing to push Docker image...'

                    // Log in to IBM Cloud using API Key
                    withCredentials([string(credentialsId: 'ibmcloud-api-key', variable: 'IBM_CLOUD_API_KEY')]) {
                        bat """
                        echo 🔐 Logging into IBM Cloud...
                        ibmcloud login --apikey %IBM_CLOUD_API_KEY% -r jp-tok
                        ibmcloud cr login
                        """
                    }

                    // Tag and Push Docker Image
                    def imageName = "${IBM_CLOUD_REGISTRY_URL}/${IBM_CLOUD_REGISTRY_NAMESPACE}/config-manage:latest"
                    bat """
                    echo 🚀 Tagging and Pushing Docker Image...
                    docker tag config-manage:latest ${imageName}
                    docker push ${imageName}
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'ibm-cloud-api-key', variable: 'IBM_CLOUD_API_KEY')]) {
                        bat """
                        echo ⚙️ Configuring Kubernetes Cluster...
                        ibmcloud ks cluster config --cluster <your-cluster-name>

                        echo 📡 Deploying to Kubernetes...
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                        """
                    }
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
