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
            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
            reuseNode true
        }
    }
    steps {
        sh '''
            apt-get update && apt-get install -y netcat-openbsd
            npm install serve
            npx serve -s build -l 5000 &
            timeout 20 bash -c "until nc -z localhost 5000; do sleep 1; done"
            echo "✅ Server is running, you can now run Playwright tests here."
        '''
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
                    npm install netlify-cli
                    npx netlify --version
                    # Add your deployment commands here
                '''
            }
        }
    }
}
