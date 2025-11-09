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
                    echo "=== STARTING STATIC SERVER ==="
                    npm install -g serve
                    serve -s build &
                    SERVER_PID=$!
                    sleep 10

                    echo "=== RUNNING PLAYWRIGHT E2E TESTS ==="
                    npx playwright test --reporter=html

                    echo "Killing server (PID: $SERVER_PID)"
                    kill $SERVER_PID || true
                '''
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Playwright E2E Report'
                    ])
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
                    echo "=== INSTALLING NETLIFY CLI ==="
                    npm install -g netlify-cli

                    echo "=== DEPLOYING TO NETLIFY ==="
                    netlify deploy --dir=build --prod --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID
                '''
            }
        }
    }
}