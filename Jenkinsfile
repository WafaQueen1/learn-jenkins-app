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
                    ls -la
                    
                    npm --version
                    npm ci
                    
                    echo "=== RUNNING BUILD ==="
                    npm run build
                    
                    echo "=== BUILD OUTPUT (build/ folder) ==="
                    ls -la build/
                    
                    echo "Build completed successfully!"
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true  // Same workspace → sees build/ folder
                }
            }
            steps {
                sh '''
                    if [ -f "build/index.html" ]; then
                        echo "PASSED: build/index.html EXISTS"
                    else
                        echo "FAILED: build/index.html NOT FOUND"
                        exit 1
                    fi
                    
                    npm test 
                '''
            }
        }
    }
    post {
        always {
            // Publish JUnit test results
            junit testResults: 'junit-results/junit.xml'
        }
    }
}