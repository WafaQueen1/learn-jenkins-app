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
                    echo "=== BUILDING PROJECT ==="
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
                    # Browsers already included in the image
                    npx playwright test --reporter=html --output=test-results/
                '''
            }
            post {
                always {
                    // Only if your tests produce JUnit XML
                    junit 'test-results/junit.xml'
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
                    echo "=== DEPLOYING ==="
                    npm install -g netlify-cli
                    netlify --version
                    # netlify deploy --dir=build --prod
                '''
            }
        }
    }
}
