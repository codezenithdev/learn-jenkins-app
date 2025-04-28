
pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'd8459211-fc31-4ff9-887b-6e8d9723c7ec'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }
        stages {
            stage('Build') {
                agent {
                    docker {
                        image 'node:18-alpine'
                        reuseNode true
                    }
                }
                steps {
                sh '''
                        ls -la
                        node --version
                        npm --version
                        npm ci
                        npm run build
                        ls -la
                   '''
                }
            }

            stage('Stage Test') {
                parallel {
                    stage('Unit Test') {
                        agent {
                            docker {
                                image 'node:18-alpine'
                                reuseNode true
                                args '-u 0'
                            }
                        }
                        steps {
                            sh '''
                                test -f build/index.html
                                npm test
                            '''
                        }
                        post {
                                always {
                                    junit 'jest-results/junit.xml'
                                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                                }
                        }
                    }

                    stage('E2E Test') {
                        agent {
                            docker {
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
                        post {
                                always {
                                    junit 'jest-results/junit.xml'
                                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                                }
                        }
                    }
                }
            }
            stage('Deploy staging') {
                agent {
                    docker {
                        image 'node:18-alpine'
                        reuseNode true
                    }
                }
                steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to production site id is: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                   '''
                }
            }
            stage('Deploy production') {
                agent {
                    docker {
                        image 'node:18-alpine'
                        reuseNode true
                    }
                }
                steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production site id is: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                   '''
                }
            }
        stage('Approval'){
            steps{
                input message: 'Do you wish to deploy to production', ok: 'Yes, I\'m sure!'
            }
        }

        stage('PostDeployment') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://serene-monstera-7863a8.netlify.app'
            }
            steps {
                sh '''
                                npx playwright test --reporter=html
                            '''
            }
            post {
                always {
                    junit 'jest-results/junit.xml'
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Paywright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        }
}
