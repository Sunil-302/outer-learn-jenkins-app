pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
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
        stage('Test') {
            steps {
            sh '''
            echo "Test existence of build/index file"
            test -f build/index.html
            grep "charset" build/index.html
            echo "Runniing test"
            npm test
            '''
            }
        }
    }
}
