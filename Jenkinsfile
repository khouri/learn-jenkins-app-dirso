pipeline {
    agent any

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
                    chown -R adilson:sudo ~/.npm
                    chown -R jenkins:sudo ~/.npm
                    npm ci
                    npm run build
                    ls -la
                    '''
            }
        }
        
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh ''' 
                    test -f build/index.html
                    npm test
                    '''
            }
        }


    }

}
