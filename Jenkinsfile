pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = credentials('netlify-id-site')
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
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

       stage('Deploy - Stage') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to production: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                   '''
                   
                script {
                    env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json",
                                         returnStdout:true)
            }
            }
        }

        stage('Staging - E2E Test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    args '-u root'
                    reuseNode true
                }
            }
            environment {
                        CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }
            steps {
                sh '''
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




        /*validacao manual*/
        stage('Approval') {
            steps{
                timeout(time: 15, unit: 'MINUTES') {
                   input message: 'Podemos fazer deploy em Prod?', ok: 'deploy'
                }
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                   '''
            }
        }

        stage('Prod E2E Test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    args '-u root'
                    reuseNode true
                }
            }
            environment {
                    CI_ENVIRONMENT_URL = 'https://tubular-tapioca-a21ef2.netlify.app'
            }
            steps {
                sh '''
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
