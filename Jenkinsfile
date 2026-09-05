pipeline {
    agent any

    environment{
        NETLIFY_SITE_ID = '1b63a054-5897-4853-a6f1-1288651ec55d'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        }

    stages {
        stage('docker'){
            steps{
                sh 'docker build -t my-playwright .'
            }
        }
        
        stage('approval to build') {
            steps {
                timeout(15) {
                    input message: 'Do you wish to procede to build?', ok: ' Yes, I am sure!'
                }
            }
        }

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
                            image 'mcr.microsoft.com/playwright:v1.62.0-noble'
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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'local e2e playwright report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('approval to staging deploy') {
            steps {
                timeout(15) {
                    input message: 'Do you wish to procede to deploy to staging?', ok: ' Yes, I am sure!'
                }
            }
        }

        stage('deploy staging') {
            agent {
                docker{
                    image 'mcr.microsoft.com/playwright:v1.62.0-noble'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = "TO BE DEFINE"    
            }
            steps {
                sh '''
                echo "Deploy to staging"
                export HOME=$WORKSPACE

                npm install netlify-cli node-jq
                npx netlify status

                mkdir -p deploy-output
                npx netlify deploy --no-build --dir=build --json > deploy-output/staging.json
                CI_ENVIRONMENT_URL=$(npx node-jq -r '.deploy_url' deploy-output/staging.json)

                echo "Test staging deploy"
                npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'staging e2e playwright report', reportTitles: '', useWrapperFileDirectly: true])
                }
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
                    image 'mcr.microsoft.com/playwright:v1.62.0-noble'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'https://starlit-panda-a47b4a.netlify.app'    
            }
            steps {
                sh '''
                echo "Deploy to prod"
                export HOME=$WORKSPACE

                npm install netlify-cli
                npx netlify status

                mkdir -p deploy-output
                npx netlify deploy --no-build --dir=build --prod --json > deploy-output/prod.json
                
                echo "Test prod deploy"
                npx playwright test --reporter=html
                '''
            }
            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'prod e2e playwright report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
