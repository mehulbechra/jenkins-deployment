pipeline {
    agent any
    
    stages {
        stage('docker') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -a
                '''
            }
        }
    }
}