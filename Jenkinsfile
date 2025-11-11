pipeline {
    agent none
   environment{
    NETLIFY_SITE_ID='c68bc995-e11b-4822-a1b0-d73ae9e53148'
   }
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
            parallel {
                stage('Junit test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'test-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.42.0-focal'
                            reuseNode true
                        }
                    }
                    steps {
    sh '''
        # Start the app with serve
        npx serve -s build -l 3000 &

        # Wait until it's ready
        until curl -s http://localhost:3000 > /dev/null; do
            echo "Waiting for app to start on port 3000..."
            sleep 1
        done
        echo "App is running!"

        # Now run Playwright — it will NOT try to start another server
        npx playwright test --reporter=html
    '''
}
                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                keepAll: false,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright E2E Report'
                            ])
                        }
                    }
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
                    npm install netlify-cli@20.1.1
                    
                    node_modules/.bin/netlify --version
                    echo "Deployment of site: $NETLIFY_SITE_ID"
                '''
            }
        }
    }
}
