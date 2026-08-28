pipeline {
    agent any

    stages {
        stage('build') {
            agent {
                docker{
                    image 'node:lts-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                ls -la
                node --version
                npm --version
                '''
                sh 'npm ci'
                sh 'npm run build'
            }
        }
    }
}
