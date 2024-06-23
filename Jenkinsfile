pipeline {
    agent any

    stages {
        /*
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
        */

        stage ('Tests') {
            parallel {

                        stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                    sh '''
                    # echo "Test existence of build/index file"
                    # test -f build/index.html
                    # grep "charset" build/index.html
                    # echo "Running test"
                    npm test
                    '''
                    }
                }

                stage('E2e') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                    sh '''
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 60
                        npx playwright test --reporter=html
                    '''
                    }
                }

            }
        }

        
    }

    post {
        always {
            junit 'jest-results/junit.xml'
        }
    }
}
