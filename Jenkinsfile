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
                sh 'echo "Build stage"'
                sh 'npm ci'
                sh 'npm run build'
            }
        }
        stage('test'){
            agent {
                docker{
                    image 'node:lts-alpine'
                    reuseNode true
                }
            }
            steps{
                sh 'echo "Test stage"'
                sh 'test -f build/index.html'
                sh 'npm test'
            }
        }
    }
}
