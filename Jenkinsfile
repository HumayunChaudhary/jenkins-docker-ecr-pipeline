
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

        stage ('Push Images to ECR') {
            steps {
                sh '''
                    docker push $ECR_REGISTRY/demo_app:frontend-$GIT_COMMIT
                    docker push $ECR_REGISTRY/demo_app:backend-$GIT_COMMIT
                '''
            }
        }

        stage('Deploy to ECS/Fargate') {
            parallel {
                stage('Deploy Frontend') {
                    steps {
                        sh '''
                            FRONTEND_IMAGE=$ECR_REGISTRY/demo_app:frontend-$GIT_COMMIT

                            aws ecs describe-task-definition \
                            --task-definition frontend-task \
                            --region $AWS_REGION \
                            --query taskDefinition > frontend-task-def.json

                            cat frontend-task-def.json | jq \
                            --arg IMAGE "$FRONTEND_IMAGE" \
                            '.containerDefinitions[0].image = $IMAGE
                            | del(.taskDefinitionArn, .revision, .status, .requiresAttributes, .compatibilities, .registeredAt, .registeredBy)' \
                            > frontend-new-task-def.json

                            FRONTEND_TASK_DEF_ARN=$(aws ecs register-task-definition \
                            --cli-input-json file://frontend-new-task-def.json \
                            --region $AWS_REGION \
                            --query 'taskDefinition.taskDefinitionArn' \
                            --output text)

                            aws ecs update-service \
                            --cluster ecs-cluster-demo-app \
                            --service frontend-service \
                            --task-definition $FRONTEND_TASK_DEF_ARN \
                            --region $AWS_REGION
                        '''
                    }
                }

                stage('Deploy Backend') {
                    steps {
                        sh '''
                            BACKEND_IMAGE=$ECR_REGISTRY/demo_app:backend-$GIT_COMMIT

                            aws ecs describe-task-definition \
                            --task-definition backend-task \
                            --region $AWS_REGION \
                            --query taskDefinition > backend-task-def.json

                            cat backend-task-def.json | jq \
                            --arg IMAGE "$BACKEND_IMAGE" \
                            '.containerDefinitions[0].image = $IMAGE
                            | del(.taskDefinitionArn, .revision, .status, .requiresAttributes, .compatibilities, .registeredAt, .registeredBy)' \
                            > backend-new-task-def.json

                            BACKEND_TASK_DEF_ARN=$(aws ecs register-task-definition \
                            --cli-input-json file://backend-new-task-def.json \
                            --region $AWS_REGION \
                            --query 'taskDefinition.taskDefinitionArn' \
                            --output text)

                            aws ecs update-service \
                            --cluster ecs-cluster-demo-app \
                            --service backend \
                            --task-definition $BACKEND_TASK_DEF_ARN \
                            --region $AWS_REGION
                        '''
                    }
                }
            }
        }
    }
}

