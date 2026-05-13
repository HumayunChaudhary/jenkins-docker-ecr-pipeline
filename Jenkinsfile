pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = credentials('aws-account-id')
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }
    stages {
        stage ('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ECR_REGISTRY
                '''
            }
        }

        stage ('Build Images')
        {
           parallel {
                stage ('Build Frontend') {
                    steps {
                        sh '''
                            cd frontend/
                            docker build -t $ECR_REGISTRY/demo_app:frontend-$GIT_COMMIT .
                        '''
                    }
                }

                stage ('Build Backend') {
                    steps {
                        sh '''
                            cd backend/
                            docker build -t $ECR_REGISTRY/demo_app:backend-$GIT_COMMIT .
                        '''
                    }
                }
           } 
        }

        stage ('Push Images') {
            steps {
                sh '''
                    docker push $ECR_REGISTRY/demo_app:frontend-$GIT_COMMIT
                    docker push $ECR_REGISTRY/demo_app:backend-$GIT_COMMIT
                '''
            }
        }
    }
}
