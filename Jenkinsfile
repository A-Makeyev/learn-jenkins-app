pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '92bef786-9da6-446c-a054-7f30bda3d912'
        NETLIFY_AUTH_TOKEN = credentials('netlify_token')
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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
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
                    test -f build/index.html
                    npm test
                '''
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
                    npm i serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test --reporter=html
                '''
            }

            // post {
            //     always {
            //         publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reporterDir: 'reports'])
            //     }
            // }
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
                    npm i netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}