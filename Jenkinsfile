pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '1b63a054-5897-4853-a6f1-1288651ec55d'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        }

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
        stage('deploy-staging') {
            agent {
                docker{
                    image 'node:lts-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo "Deploy to staging stage"
                export HOME=$WORKSPACE

                npm install netlify-cli
                npx netlify status

                npx netlify deploy --no-build --dir=build
                '''
            }
        }
        stage('approval') {
            steps {
                timeout(15) {
                    input message: 'Do you wish to deploy to production?', ok: ' Yes, I am sure!'
                }
            }
        }
        stage('deploy-prod') {
            agent {
                docker{
                    image 'node:lts-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo "Deploy to prod stage"
                export HOME=$WORKSPACE

                npm install netlify-cli
                npx netlify status

                npx netlify deploy --no-build --dir=build --prod
                '''
            }
        }
        stage('prod-e2e') {
            agent {
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'https://starlit-panda-a47b4a.netlify.app'    
            }
            steps {
                sh '''
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
