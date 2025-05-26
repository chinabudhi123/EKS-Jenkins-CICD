pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        EKS_CLUSTER_NAME = "your-eks-cluster-name" // ✅ Replace with your EKS cluster name
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials') // Jenkins credentials ID
        DOCKERHUB_IMAGE = "${DOCKERHUB_CREDENTIALS_USR}/myapp:latest"
    }

    stages {
        stage('Checkout from GitHub') {
            steps {
                git branch: 'main', url: 'https://github.com/satya44jit/eks-jenkins-cicd.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t "$DOCKERHUB_IMAGE" .'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                sh '''
                    echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh 'docker push "$DOCKERHUB_IMAGE"'
            }
        }

        stage('Update Kubeconfig') {
            steps {
                sh 'aws eks update-kubeconfig --region "$AWS_REGION" --name "$EKS_CLUSTER_NAME"'
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    echo "Updating image in deployment YAML..."
                    sed -i "s|image:.*|image: ''' + "${DOCKERHUB_IMAGE}" + '''|" k8s/deployment.yaml

                    echo "Deploying to EKS..."
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD pipeline executed successfully and app is deployed to EKS!'
        }
        failure {
            echo '❌ Pipeline failed. Please check the logs for more information.'
        }
    }
}

