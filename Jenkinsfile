pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = credentials('netlify-id-site')
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.2.$BUILD_ID"
    }

    stages {
        /* moved to another pipeline
        stage('Docker') {
            steps {
                sh 'docker buildx build -t my-playwright --file Dockerfile .'
            }
        }
        */
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root'
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

        stage('Group Test') {
            parallel {
                    stage('Test') {
                        agent {
                            docker {
                                image 'node:18-alpine'
                                args '-u root'
                                reuseNode true
                            }
                        }
                        steps {
                            sh '''
                                test -f build/index.html
                                npm test
                                ls -la
                                '''
                        }
                        post {
                                always {
                                    junit '**/jest-results/*.xml'
                                }
                        }
                    }

                    stage('E2E') {
                        agent {
                            docker {
                                image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                                args '-u root'
                                reuseNode true
                            }
                        }
                        steps {
                            sh '''
                                npm install serve
                                node_modules/.bin/serve -s build &
                                sleep 10
                                npx playwright test --reporter=html
                            '''
                        }
                    post {
                            always {
                                    publishHTML([allowMissing: false,
                                        alwaysLinkToLastBuild: false,
                                        icon: '',
                                        keepAll: false,
                                        reportDir: 'playwright-report',
                                        reportFiles: 'index.html',
                                        reportName: 'HTML Report',
                                        reportTitles: '',
                                        useWrapperFileDirectly: true])
                            }
                    }
                    }
            }
        }

        stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-playwright'
                    args '-u root'
                    reuseNode true
                }
            }
            environment {
                        CI_ENVIRONMENT_URL = "TO_BE_DEFINED_LATTER"
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                    npx playwright test --reporter=html
                    '''
            }
            post {
                always {
                    publishHTML([allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        icon: '',
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'HTML Report E2E Staging',
                        reportTitles: '',
                        useWrapperFileDirectly: true])
                }
            }
        }

        /*validacao manual
        stage('Approval') {
            steps{
                timeout(time: 15, unit: 'MINUTES') {
                   input message: 'Podemos fazer deploy em Prod?', ok: 'deploy'
                }
            }
        }
        */

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playwright'
                    args '-u root'
                    reuseNode true
                }
            }
            environment {
                    CI_ENVIRONMENT_URL = 'https://tubular-tapioca-a21ef2.netlify.app'
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to production: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                    '''
            }
            post {
                always {
                    publishHTML([allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        icon: '',
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'HTML Report Prod',
                        reportTitles: '',
                        useWrapperFileDirectly: true])
                }
            }
        }
    }
}
