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
                    image 'mcr.microsoft.com/playwright:v1.42.0-focal'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "=== RUNNING PLAYWRIGHT TESTS ==="
                    npm ci
                    npx playwright install
                    npx playwright test --reporter=html --output test-results/
                '''
            }
            post {
                always {
                    junit 'test-results/junit.xml' // Make sure your Playwright tests generate JUnit XML
                }
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
                    node_modules/.bin/netlify --version
                    # Add deployment command here
                '''
            }
        }
    }
}
