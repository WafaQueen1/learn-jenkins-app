pipeline {
    agent any

    stages {
        stage('Build') {
            agent{
                docker {
                image 'node:18-alpine'
                reuseNose true
               }
            }  
            steps {
                sh'''
                ls -la
                npm --version
                npm ci
                npm run build
                ls -la
                '''
            }         
        }
    }
}
