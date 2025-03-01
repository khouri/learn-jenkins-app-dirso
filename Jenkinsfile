pipeline {
    agent any

    environment {
                    NETLIFY_SITE_ID = eef27ec1-6099-425b-b0ea-7a9dd3fb8f5e
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

        stage('Group Test'){
            parallel{
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
                                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }
                    }
            }
        }

        stage('Deploy') {

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
                   '''
            }
        }

    }
}
