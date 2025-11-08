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
    }
}  