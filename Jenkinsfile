pipeline {
    agent none  // Correct: let each stage define its agent

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
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "=== RUNNING TESTS ==="
                    
                    if [ -f "build/index.html" ]; then
                        echo "PASSED: build/index.html EXISTS"
                    else
                        echo "FAILED: build/index.html NOT FOUND"
                        exit 1
                    fi
                    
                    npm test -- --reporters=default --reporters=jest-junit
                '''
            }
        }
    }

    post {
        always {
            node('any') {  // REQUIRED: junit needs a node
                junit testResults: 'junit-results/junit.xml', allowEmptyResults: true
            }
        }
    }
}