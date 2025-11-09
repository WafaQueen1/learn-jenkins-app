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
                    image 'mcr.microsoft.com/playwright:v1.42.0-focal' // prebuilt Playwright image
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "=== TESTING INSIDE DOCKER ==="
                    npm ci
                    npx playwright install  # installs browsers
                    node_modules/.bin/serve -s build & 
                    sleep 10
                    npx playwright test --reporter=junit --output test-results
                '''
            }
            post{
                always{
                     junit 'test-results/*.xml'
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
                    # deploy command example:
                    # node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
    }
}
