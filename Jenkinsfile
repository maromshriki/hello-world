pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1' 
        ECR_REPO_NAME = 'marom1'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        AWS_ACCOUNT_ID = '992382545251'
        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: 'https://github.com/maromshriki/hello-world.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${ECR_REPO_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-ecr-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set default.region ${AWS_REGION}

                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_URI}
                    '''
                }
            }
        }

        stage('Tag & Push Image to ECR') {
            steps {
                script {
                    sh """
                        docker tag ${ECR_REPO_NAME}:${IMAGE_TAG} ${ECR_URI}:${IMAGE_TAG}
                        docker push ${ECR_URI}:${IMAGE_TAG}
                    """
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

