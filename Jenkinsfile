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
                npm ci --no-bin-links --cache .npm-cache
                npm run build
                '''
            }
        }
    }
}
