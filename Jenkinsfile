pipeline {
    agent any

    stages {
        stage('build') {
            agent {
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                ls -la
                node --version
                npm --version
                '''
                sh 'rm -rf node_modules .npm-cache'
                sh 'npm ci'
                sh 'npm run build'
            }
        }
    }
}
