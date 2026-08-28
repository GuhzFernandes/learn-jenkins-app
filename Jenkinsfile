pipeline {
    agent any

    stages {
        stage('build') {
            agent {
                docker{
                    image 'node:lts-alpine'
                    reuseNode true
                    args '-e NPM_CONFIG_CACHE=${WORKSPACE_TMP}/.npm-cache'
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
