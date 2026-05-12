pipeline {
    agent any
    stages {
        stage ('Frontend image building') {
            steps {
                sh '''
                    cd frontend/
                    docker build -t humayun27/frontend:$GIT_COMMIT .
                '''
            }
        }

        stage ('Backend image building') {
            steps {
                sh '''
                    cd backend/
                    docker build -t humayun27/backend:$GIT_COMMIT .
                '''
            }
        }
    }
}
