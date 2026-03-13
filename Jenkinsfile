pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        TERRAFORM_DIR = 'terraform'
        HELM_DIR = 'helm'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/yourusername/your-microservices-repo.git', branch: 'main'
            }
        }

        stage('Build Microservices') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'aws-creds',
                                  usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin 123456789012.dkr.ecr.$AWS_REGION.amazonaws.com

                        docker build -t user-service ./user-service
                        docker tag user-service:latest 123456789012.dkr.ecr.$AWS_REGION.amazonaws.com/user-service:latest
                        docker push 123456789012.dkr.ecr.$AWS_REGION.amazonaws.com/user-service:latest
                    '''
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir("${TERRAFORM_DIR}") {
                    sh 'terraform init'
                    sh 'terraform apply -auto-approve'
                }
            }
        }

        stage('Helm Deploy') {
            steps {
                sh 'aws eks --region $AWS_REGION update-kubeconfig --name $(terraform output -raw eks_cluster_name)'

                sh "helm upgrade --install user-service ${HELM_DIR}/user-service --set image.repository=123456789012.dkr.ecr.$AWS_REGION.amazonaws.com/user-service,image.tag=latest"
            }
        }
    }
}
