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
        stage('tests'){
            parallel{
                stage('junit'){
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
                    post{
                        always{
                            junit 'junit-test-results/junit.xml'
                        }
                    }
                }
                stage('e2e') {
                    agent {
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                        npm install serve
                        npx serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage('deploy') {
            agent {
                docker{
                    image 'node:lts-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo "Deploy stage"
                export HOME=$WORKSPACE
                export USER=jenkins

                npm install netlify-cli@20.1.1
                npx netlify --version
                '''
            }
        }
    }
}
