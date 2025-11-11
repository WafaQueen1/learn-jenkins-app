pipeline {
    agent none
    environment {
        NETLIFY_SITE_ID = 'c68bc995-e11b-4822-a1b0-d73ae9e53148'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    options {
        timeout(time: 15, unit: 'MINUTES')
        retry(2)  // retry whole pipeline on network failure
    }

    stages {
        /* =============================================
           BUILD – Fast + Network Resilient
           ============================================= */
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                retry(3) {
                    timeout(time: 8, unit: 'MINUTES') {
                        sh '''
                            echo "=== BUILDING (with retry) ==="
                            # Use fast mirror (China) – works great in DZ
                            npm config set registry https://registry.npmmirror.com
                            npm config set fetch-retries 5
                            npm config set fetch-retry-mintimeout 20000
                            npm config set fetch-retry-maxtimeout 90000

                            # Cache node_modules
                            npm ci
                            npm run build
                            ls -la build/
                        '''
                    }
                }
            }
        }

        /* =============================================
           TEST – Parallel + Cached
           ============================================= */
        stage('Test') {
            parallel {
                stage('JUnit') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh 'npm test'
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
                            npx serve -s build -l 3000 &
                            until curl -s http://localhost:3000 >/dev/null; do
                                echo "Waiting for app..."
                                sleep 1
                            done
                            echo "App running!"
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

        /* =============================================
           DEPLOY – Fast with npx (no install!)
           ============================================= */
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-bullseye'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "=== DEPLOYING TO NETLIFY ==="
                    echo "Site ID: $NETLIFY_SITE_ID"

                    # Use npx → no npm install → no sharp compile → < 30 sec
                    npx netlify-cli@20.1.1 --version
                    npx netlify-cli@20.1.1 status

                    echo "Deploying build/ to production..."
                    npx netlify-cli@20.1.1 deploy \
                        --dir=build \
                        --prod \
                        --site=$NETLIFY_SITE_ID
                '''
            }
        }
    }

    post {
    success {
        echo "Pipeline SUCCEEDED! Site deployed: https://${NETLIFY_SITE_ID}.netlify.app"
    }
    failure {
        echo "Pipeline FAILED. Check logs."
    }
}
}