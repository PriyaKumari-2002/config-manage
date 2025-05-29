pipeline {
    agent any

    environment {
        PATH = "C:\\Program Files\\Docker\\Docker\\resources\\bin;C:\\Program Files\\IBM\\Cloud\\bin;C:\\Windows\\System32"
        IBM_CLOUD_REGION = 'ap-north'
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

                    withCredentials([string(credentialsId: 'ibmcloud-api-key', variable: 'IBM_CLOUD_API_KEY')]) {
                        bat """
                        echo 🔐 Setting IBM Cloud API Endpoint...
                        ibmcloud api https://cloud.ibm.com

                        echo 🔐 Logging into IBM Cloud...
                        ibmcloud login --apikey ${IBM_CLOUD_API_KEY} -r ${IBM_CLOUD_REGION}
                        ibmcloud cr login
                        ibmcloud cr region-set ${IBM_CLOUD_REGION}
                        """
                    }

                    def imageName = "${IBM_CLOUD_REGISTRY_URL}/${IBM_CLOUD_REGISTRY_NAMESPACE}/config-manage:latest"
                    bat """
                    echo 🚀 Tagging and Pushing Docker Image...
                    docker tag config-manage:latest ${imageName}
                    docker push ${imageName}
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
