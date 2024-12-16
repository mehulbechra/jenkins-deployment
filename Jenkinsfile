pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '3c7a6f2f-b3e2-48e1-9444-791a1804abc2'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
    
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm ci
                    npm run build
                '''
            }
        }
        stage('Run Tests') {
            parallel {
                stage('Test') {
                    agent {
                        docker {
                            image 'node:20-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            test -f build/index.html
                            npm run test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm i serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy Stage') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm i netlify-cli node-jq
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r ".deploy_url" deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Stag', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Approval') {
            steps {
                timeout(time:15, unit:'MINUTES') {
                    input 'Ready to deploy?'
                }
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL='https://dulcet-smakager-525d8e.netlify.app'
            }
            steps {
                sh '''
                    npm i netlify-cli
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}