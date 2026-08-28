pipeline {
    agent any

    stages {
        stage('build') {
            agent {
                docker{
                    image 'node:lts-alpine'
                    reuseNode true
                    args '-u jenkins:jenkins'
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
