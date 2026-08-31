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
        stage('e2e') {
            agent {
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-noble'
                    reuseNode true
                }
            }
            steps {
                sh '''
                npm install serve
                node_modules/.bin/serve -s build &
                sleep 10
                npx playwright test
                '''
            }
        }
    }
    post{
        always{
            junit 'test-results/junit.xml'
        }
    }
}
