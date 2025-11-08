pipeline {
    agent none  // Best practice: let each stage define its agent

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true  // ← Fixed typo: reuseNose → reuseNode
                }
            }
            steps {
                sh '''
                    echo "=== BUILDING INSIDE DOCKER ==="
                    ls -la
                    npm --version
                    npm ci
                    npm run build
                    echo "=== BUILD OUTPUT ==="
                    ls -la dist/  # Show built files
                '''
            }
        }
    }

    
}