pipeline {
    agent any

    environment {
      NETLIFY_SITE_ID = '2fbd0171-f68f-40df-8e55-cc2d8bf8dd24'
      NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        /*

        stage('Up Docker image') {
            steps {
                sh '''
                docker build -t netlify-playwright .
                '''
            }
        }

        */

        stage('Build') {
            agent {
                docker {
                    image 'netlify-playwright'
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

        stage('Tests') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'netlify-playwright'
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
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2e') {
                    agent {
                        docker {
                            image 'netlify-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''                        
                        serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        

        stage('DeployProd & test E2E') {
            agent {
                docker {
                    image 'netlify-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://jade-puffpuff-078189.netlify.app/'
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod --json > deploy-output.json
                    node-jq -r '.deploy_url' deploy-output.json
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}
