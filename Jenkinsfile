pipeline {
    agent any
    stages {
       
        stage ('Docker test') {
            steps {
                sh 'docker ps > testfile.txt'
            }
        }
    }
}
