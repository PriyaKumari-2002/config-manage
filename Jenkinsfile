pipeline {
    agent any

    environment {
        PATH = "C:\\Windows\\System32"
        DOCKER_HOST = 'tcp://localhost:2375'  // Use the Docker API over TCP if required
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
                    docker.build("config-manage:latest", "--file=docker/Dockerfile .")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    sh '"C:\\Windows\\System32\\cmd.exe" /c echo Hello'
                    sh "ibmcloud login --apikey $IBM_CLOUD_API_KEY -r us-south"
                    sh "ibmcloud cr login"

                    def imageName = "${IBM_CLOUD_REGISTRY_URL}/${IBM_CLOUD_REGISTRY_NAMESPACE}/config-manage:latest"
                    sh "docker tag config-manage:latest ${imageName}"
                    sh "docker push ${imageName}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "ibmcloud ks cluster config --cluster <your-cluster-name>"
                    sh "kubectl apply -f k8s/deployment.yaml"
                    sh "kubectl apply -f k8s/service.yaml"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
