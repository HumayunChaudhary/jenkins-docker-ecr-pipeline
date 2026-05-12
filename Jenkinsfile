pipeline {
    agent any
    stages {
        stage ('Test stage') {
            steps {
                sh '''
                    echo "This is Jenkinsfile" > testfile.txt
                '''
            }
        }
    }
}
