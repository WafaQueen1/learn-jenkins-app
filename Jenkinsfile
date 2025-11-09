pipeline {
    agent none

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
                    echo "=== BUILDING INSIDE DOCKER ==="
                    npm ci
                    npm run build
                    ls -la build/
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
                    echo "=== RUNNING TESTS ==="
                    
                    # Check build output
                    [ -f build/index.html ] && echo "PASSED: build/index.html EXISTS"
                    
                    # Run tests with JUnit output
                    npm test -- --reporters=default --reporters=jest-junit
                '''
            }
            
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install -g netlify-cli
                    netlify --version
                '''
            }
        }

    }
}